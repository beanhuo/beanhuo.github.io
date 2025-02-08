---
layout: post
title: Django: messages.success(), messages.error(), and JsonResponse()
date: 2015-01-20 
tags: Django   
---


# Django: messages.success(), messages.error(), and JsonResponse()
--------

　In Django, there are multiple ways to send feedback or data from the backend to the frontend. This post explains the differences between messages.success(), messages.error(), and JsonResponse(), along with examples for each.

## 1. messages.success() and messages.error()

* Purpose
    * Part of Django's messaging framework.

    * Used to send one-time notifications (e.g., success or error messages) to the user after a form submission or other actions.

    * Messages are stored in the session and displayed on the next page the user visits.

* When to Use
Use messages.success() and messages.error() when:

You want to display a message after a page reload or redirect.

You are not using JavaScript for form submission (e.g., traditional form submission with a page reload).

* Example
Backend

```
from django.contrib import messages
from django.shortcuts import redirect, render

def submit_handler(request):
    if request.method == 'POST':
        try:
            # Process the form data
            # ...

            # Send a success message
            messages.success(request, "You have successfully submitted the log. You can now manage it.")
            return redirect('lioscope_app:iokpp-submit')  # Redirect to the same page
        except Exception as e:
            # Send an error message
            messages.error(request, f"An error occurred: {e}")
            return redirect('lioscope_app:iokpp-submit')  # Redirect to the same page
```
Frontend (Template)
```
{% if messages %}
    <div class="messages">
        {% for message in messages %}
            <div class="alert {% if message.tags %}alert-{{ message.tags }}{% endif %}">
                {{ message }}
            </div>
        {% endfor %}
    </div>
{% endif %}


```  
Frontend (JavaScript)


```
const xhr = new XMLHttpRequest();
xhr.open("POST", "/submit/", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

xhr.onreadystatechange = function () {
    if (xhr.readyState === XMLHttpRequest.DONE) {
        if (xhr.status === 200) {
            const response = JSON.parse(xhr.responseText);
            if (response.status === "success") {
                alert(response.message);  // Display success message
            } else if (response.status === "error") {
                alert(response.message);  // Display error message
            }
        } else {
            alert("An unexpected error occurred. Please try again.");
        }
    }
};

xhr.send(new URLSearchParams(new FormData(form)));

```





## 2. JsonResponse()

Purpose
Used to return JSON-encoded responses from the backend to the frontend.

Commonly used in AJAX-based applications where the frontend (JavaScript) handles the response dynamically without reloading the page.

When to Use
Use JsonResponse when:

You are using JavaScript (e.g., XMLHttpRequest or fetch) to submit forms or make requests.

You want to update the UI dynamically without reloading the page.

You need to send structured data (e.g., status, message, or additional data) to the frontend.

Example


Backend

```
from django.http import JsonResponse

def submit_handler(request):
    if request.method == 'POST':
        try:
            # Process the form data
            # ...

            # Return a success response
            return JsonResponse({
                "status": "success",
                "message": "You have successfully submitted the log. You can now manage it."
            })
        except Exception as e:
            # Return an error response
            return JsonResponse({
                "status": "error",
                "message": str(e)  # Include the error message
            }, status=400)  # Use an appropriate HTTP status code (e.g., 400 for bad request)
```


When to Use Which?
Use messages.success() / messages.error()
When you want to display a message after a page reload or redirect.

When you are not using JavaScript for form submission.

Example: Traditional form submissions where the user is redirected to a new page.

Use JsonResponse()
When you want to update the UI dynamically without reloading the page.

When you are using JavaScript (e.g., XMLHttpRequest or fetch) for form submission.

Example: AJAX-based form submissions where the response is handled by JavaScript.

Combining Both Approaches
In some cases, you might want to use both approaches. For example:

Use JsonResponse for AJAX-based form submissions.

Use messages.success() / messages.error() as a fallback for non-JavaScript users.


```
def submit_handler(request):
    if request.method == 'POST':
        try:
            # Process the form data
            # ...

            if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
                # Return JSON response for AJAX requests
                return JsonResponse({
                    "status": "success",
                    "message": "You have successfully submitted the log. You can now manage it."
                })
            else:
                # Return a redirect with a message for non-AJAX requests
                messages.success(request, "You have successfully submitted the log. You can now manage it.")
                return redirect('lioscope_app:iokpp-submit')
        except Exception as e:
            if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
                return JsonResponse({
                    "status": "error",
                    "message": str(e)
                }, status=400)
            else:
                messages.error(request, f"An error occurred: {e}")
                return redirect('lioscope_app:iokpp-submit')

```


```
const xhr = new XMLHttpRequest();
xhr.open("POST", "/submit/", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");  // Indicate this is an AJAX request

xhr.onreadystatechange = function () {
    if (xhr.readyState === XMLHttpRequest.DONE) {
        if (xhr.status === 200) {
            const response = JSON.parse(xhr.responseText);
            if (response.status === "success") {
                alert(response.message);  // Display success message
            } else if (response.status === "error") {
                alert(response.message);  // Display error message
            }
        } else {
            alert("An unexpected error occurred. Please try again.");
        }
    }
};

xhr.send(new URLSearchParams(new FormData(form)));


```

