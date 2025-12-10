# todo_project/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    # Admin interface is always useful
    path('admin/', admin.site.urls), 
    
    # Send any requests going to the root URL (e.g., http://127.0.0.1:8000/) 
    # to the todo_app's urls.py file
    path('', include('todo_app.urls')), 
]

# todo_app/models.py

from django.db import models

class TodoItem(models.Model):
    # CharField is for short text strings
    content = models.CharField(max_length=200)
    
    # DateTimeField with auto_now_add=True saves the creation time automatically
    date_created = models.DateTimeField(auto_now_add=True)
    
    # Optional: A nice string representation for the admin panel
    def __str__(self):
        return self.content

        # todo_app/urls.py

from django.urls import path
from . import views

urlpatterns = [
    # URL: / (root) -> calls views.index function -> name='index' for use in templates
    path('', views.index, name='index'), 
    
    # URL: /add_item/ -> calls views.add_item function
    path('add_item/', views.add_item, name='add_item'),
    
    # URL: /delete_item/X (where X is the item ID) -> calls views.delete_item
    path('delete_item/<int:item_id>/', views.delete_item, name='delete_item'),
]

# todo_app/views.py

from django.shortcuts import render, redirect
from .models import TodoItem
from django.views.decorators.http import require_POST

# --- View to display the To-Do list ---
def index(request):
    # Fetch all TodoItem objects, ordered by newest first (descending date_created)
    all_todo_items = TodoItem.objects.all().order_by('-date_created') 
    
    # Context dictionary to pass data to the template
    context = {'all_items': all_todo_items}
    
    # Render the todo_list.html template with the items
    return render(request, 'todo_app/todo_list.html', context)

# --- View to add a new item (Only accepts POST requests from the form) ---
@require_POST
def add_item(request):
    # Get the data from the form field named 'content'
    content_text = request.POST['content']
    
    # Create a new TodoItem object and save it to the database
    new_item = TodoItem(content=content_text)
    new_item.save()
    
    # Redirect the user back to the main list page
    return redirect('index')

# --- View to delete an item ---
def delete_item(request, item_id):
    # Get the specific item by its primary key (ID) or return 404 if not found
    try:
        item_to_delete = TodoItem.objects.get(id=item_id)
        # Delete the item
        item_to_delete.delete()
    except TodoItem.DoesNotExist:
        # Handle case where item doesn't exist, though typically it should
        pass

    # Redirect back to the main list page
    return redirect('index')

    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple To-Do List</title>
</head>
<body>
    <h1>My Django To-Do List</h1>
    
    <form action="{% url 'add_item' %}" method="post">
        {% csrf_token %} 
        <input type="text" name="content" placeholder="Add a new to-do item" required>
        <button type="submit">Add</button>
    </form>
    
    <hr>
    
    <h2>Current Tasks:</h2>
    
    {% if all_items %}
        <ul>
        {% for todo_item in all_items %}
            <li>
                {{ todo_item.content }} 
                (Created: {{ todo_item.date_created|date:"M d, Y H:i" }})
                
                <a href="{% url 'delete_item' todo_item.id %}">
                    <button>Delete</button>
                </a>
            </li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No items in the list. Start adding some!</p>
    {% endif %}

</body>
</html>

