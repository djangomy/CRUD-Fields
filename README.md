# CRUD-Fields Varios tipos de Fields com Django 

Nesse tutorial vou mostrar para vocês os Field no Django. Vamos fazer um formulário de cadastro para conhecer alguns tipos de Field.

[*https://docs.djangoproject.com/en/4.1/ref/models/fields/*](https://docs.djangoproject.com/en/4.1/ref/models/fields/)

quando definimos:

`code_key = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)`
Como **primary_key** esse campo passa a ser nossa chave `primary_key.` Id não existe mais.

***default=uuid.uuid4*** ira gerar automaticamente uma *key*. E *editable* definido igual a *False* significa que não é um campo editável. 

Temos vários tipos de campos nesse modelo. O diferente que não pertende a biblioteca nativa do Django é o **MultiSelectField. 

`pip install django-multiselectfield`**

***myapp/models.py***

```python
import uuid
from django.db import models
from multiselectfield import MultiSelectField

# Create your models here.
CHOICES=[('SIM','SIM'),('NÃO','NÃO')]

SELECT_MULTIPLE_OPTION=(('Fever', 'Fever'),
                        ('Cough', 'Cough'),
                        ('Pain of Throat', 'Pain of Throat'),
                        ('Difficulty breathing', 'Difficulty breathing'),
                        ('Headache (Headache)', 'Headache (Headache)'),
                        ('Coryza', 'Coryza'),
                        ('I DONT HAVE SYMPTOMS', 'I DONT HAVE SYMPTOMS'),
)

class YearInSchool(models.TextChoices):
    FRESHMAN = 'FR', 'Freshman'
    SOPHOMORE = 'SO', 'Sophomore'
    JUNIOR = 'JR', 'Junior'
    SENIOR = 'SR', 'Senior'
    GRADUATE = 'GR', 'Graduate'

def contact_default():
    return {"email": "to1@example.com"}

class TestField(models.Model):
    code_key = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    code = models.IntegerField()
    date_on = models.TimeField(auto_now_add=True)
    date_public = models.DateField(null=True)
    date_time_public = models.DateTimeField(null=True)
    first_name = models.CharField(max_length=150) 
    email = models.EmailField(max_length=100)
    check_on = models.CharField(max_length=15, choices=CHOICES)
    description=models.TextField()
    user_status = MultiSelectField(max_length=200, choices=SELECT_MULTIPLE_OPTION)  
    url_link = models.URLField(max_length=200)
    image = models.ImageField('Images', upload_to='images/')
    file_item = models.FileField('Files', upload_to='Files/%Y-%m-%d/')
    json_on = models.JSONField(default=contact_default)
    float_on = models.FloatField()
    year_in_school = models.CharField(max_length=2,choices=YearInSchool.choices,default=YearInSchool.FRESHMAN,) 
    
    def __str__(self):
        return self.first_name
```

***myapp/forms.py***

```python
from django import forms
from .models import *
 
# Create the form class.
class TestFieldForm(forms.ModelForm):
    date_public = forms.DateField(widget=forms.DateInput(attrs={'type':'date'}))
    date_time_public = forms.DateTimeField(widget=forms.DateTimeInput(attrs={'type':'date'}))
    
    class Meta:
        model = TestField
        fields = '__all__' 
        
    def __init__(self, *args, **kwargs): # Adiciona 
        super().__init__(*args, **kwargs)  
        for field_name, field in self.fields.items():   
            if field.widget.__class__ in [forms.CheckboxInput, forms.RadioSelect]:
                field.widget.attrs['class'] = 'form-check-input'
            else:
                field.widget.attrs['class'] = 'form-control' 
        ## MultiSelectField campo user_status.
        self.fields['user_status'].widget.attrs['class'] = ''
```

***myapp/templates/views.py***

```python
from django.shortcuts import render, redirect

from myapp.forms import TestFieldForm
from myapp.models import TestField

# Create your views here.
def mysite(request):
    form = TestFieldForm()
    test_field = TestField.objects.all()
    if request.method == "POST":
        form = TestFieldForm(request.POST or None, request.FILES or None)
        if form.is_valid():
            form.save()
            return redirect('mysite')
    return render(request, 'index.html', {'form': form,'test_field':test_field})
```

***myapp/templates/index.html***

```html
{% extends 'base.html' %}

{% block title %}CRUDFields{% endblock %}

{% block content %} 
	<div class="container-fluid">  
		<div class="d-flex gap-3">

			<div class="bg-light p-3">
				<h1>CRUDFields</h1>
				<form class="row" method="POST" action="{% url 'mysite' %}" enctype="multipart/form-data">
					{% csrf_token %}
					
					{% for el in form %}
						<div class="col-md-4 mt-3">
							{{el.label}}
							{{el}}
						</div>
					{% endfor %} 
		
					<input class="btn btn-success mt-3" type="submit" value="save">
				</form>
			</div>

			<div class="p-5">
				<h1>Lista Cadastro</h1>
				<table class="table">

					<thead> 
						<th>Nome</th>
						<th>Code</th>
						<th>Data</th>
						<th>Check</th>
						<th>E-mail</th>
					</thead>
				
					<tbody>
						
						{% for el  in test_field %}
						<tr>  
							<td>{{el.first_name}}</td>
							<td>{{el.code}}</td>
							<td>{{el.date_public|date:"d/m/Y"}}</td> 
							<td>{{el.check_on}}</td>
							<td>{{el.email}}</td>
						</tr> 
						{% endfor %}

					</tbody>
					 
				</table>

			</div>
 
		</div> 
	</div>
	

{% endblock %}
```
