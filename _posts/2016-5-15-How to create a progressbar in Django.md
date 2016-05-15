---
layout: post
title: How to create a progressbar in Django
published: true
---

In my latest project, I created a Django website and use it to process long tasks for users, a problem is that the process time is loooong and we need a progress bar so that the users won't think the website is dead or so. After some googling, I found [Celery](http://www.celeryproject.org), which is an asynchronous task queue/job queue based on distributed message passing. Thus, I was able to achieve these functionalities for my website:

1. Start a worker in Celery to procress the long task;
2. Read message from Celery about the progress of my task;
3. Pass the message to Javescript and show the progress with a progress bar.

Although Celery is a neat and Convinient framework, its documentation is not that friendly, I've spent the last few days bumping my head to the wall. So in this post, I will write about what I learned and release a demo code, hoping my life will be easier next time I met the similar problems. For the impatient, [here](https://github.com/sunshineatnoon/Django-Celery-Example) is my demo on GitHub.

# Dependencies

* Django 1.9
* RabbitMQ 3.5.6
* Celery 3.1.19

# Prerequisites

First of all, you need a Django project, mine is called `celery_try`.

```
django-admin startproject celery_try
python manage.py makemigrations
python manage.py migrate
```

Then add those lines in your settings.py file:

```
BROKER_URL = 'amqp://guest:guest@localhost//'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
```

Those lines tell Celery we will use RabbitMQ as our message broker and we accpet json in our broker. Next create a file 'celery.py' in the folder of our website and input this:

```
from __future__ import absolute_import
import os
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'celery_try.settings')

from django.conf import settings
from celery import Celery

app = Celery('celery_try',
             backend='amqp',
             broker='amqp://guest@localhost//')

# This reads, e.g., CELERY_ACCEPT_CONTENT = ['json'] from settings.py:
app.config_from_object('django.conf:settings')

# For autodiscover_tasks to work, you must define your tasks in a file called 'tasks.py'.
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

@app.task(bind=True)
def debug_task(self):
    print("Request: {0!r}".format(self.request))
```

Those lines tell Celery that we will use the settings in setting.py we just defined.

# Tasks in Celery

Now we can start an app and define some tasks:

```
django-admin startapp testapp
```

Creat a file called tasks.py in your app's folder, all tasks shall be defined in this file. Here for demo purpose, I will define a dumy task which calculates n times discrete Fourier Transforms:

```
from celery import shared_task,current_task
from numpy import random
from scipy.fftpack import fft

@shared_task
def fft_random(n):
    """
    Brainless number crunching just to have a substantial task:
    """
    for i in range(n):
        x = random.normal(0, 0.1, 2000)
        y = fft(x)
        if(i%30 == 0):
            process_percent = int(100 * float(i) / float(n))
            current_task.update_state(state='PROGRESS',
                                      meta={'process_percent': process_percent})
    return random.random()
```

Above code is self expalainary except for this line:

```
current_task.update_state(state='PROGRESS',meta={'process_percent': process_percent})
```

This is actually a custom state in Celery, by define this state, we are able to pass the percentage of task we have processed to the message broker every 30 iterations, thus we can use this message and create a progress bar. More details about custom state can be found on [Celery documentation](http://docs.celeryproject.org/en/latest/userguide/tasks.html#custom-states). One thing to notice is that you don't want to pass too many messages to your broker, otherwise it may gets blocked, so we only pass a message every 30 iterations.

# Call tasks in Django

Here I created a page index.html that includes a submit button, when the user enters a number (say 10000) and clicks the "submit" button, Celery will call one of its workers and say: "Hey buddy, do 10000 times discrete Fourier Transforms and report the progress to me!". So how do we call Celery worker in Django? Well, this maybe the simplest part of this project, just use these two lines:

```
from .tasks import fft_random
job = fft_random.delay(int(n))
```

Now the happy worker will do this task at backend.

# Progress Bar

Now the worker is working, and in the fft_random tasks we report the percentage of task progress every 30 iterations. But how do we show it in the progress bar? We will use javescript! So in my `show_t.html` file, which is a pages shows progress of the task, I defined a javascript polling function like this:

```
<script type="text/javascript">
   var poll_xhr;
   var willstop = 0;
  (function(){
    var poll = function(){
      var json_dump = "{{ data }}";
      var task_id = "{{task_id}}";

      console.log(task_id);
      poll_xhr = $.ajax({
        url:'poll_state',
        type: 'POST',
        data: {
            task_id: task_id,
            csrfmiddlewaretoken: "{{csrf_token}}",
        },
        success: function(result) {
                    if (result.process_percent == null || result.process_percent == undefined) {
                        willstop = 1;
                        document.getElementById("user-count").textContent="DONE";
                        jQuery('.bar').css({'width': 100 + '%'});
                        jQuery('.bar').html(100 + '%');
                        document.getElementById('returnBtn').style.visibility = 'visible';

                       } else {
                         jQuery('.bar').css({'width': result.process_percent + '%'});
                         jQuery('.bar').html(result.process_percent + '%');
                         document.getElementById("user-count").textContent="PROCRESSING";
                       };
                    }
      });
    };

    var refreshIntervalId = setInterval(function() {
      poll();
      if(willstop == 1){
        clearInterval(refreshIntervalId);
      }
    },500);


  })();
</script>
```

Long story short, this piece javascript code grab  `task_id` from Django and use send this `task_id` through a POST request to a Django view call `poll_state`. This `poll_state` function checks task's state and return them in a json string to javascript. Then javascript can do whatever it wants with the status: showing status, update a progress bar ... Here is how the `poll_state` function in `views.py`:

```
# Create your views here.
def poll_state(request):
    """ A view to report the progress to the user """
    data = 'Fail'
    if request.is_ajax():
        if 'task_id' in request.POST.keys() and request.POST['task_id']:
            task_id = request.POST['task_id']
            task = AsyncResult(task_id)
            data = task.result or task.state
        else:
            data = 'No task_id in the request'
    else:
        data = 'This is not an ajax request'

    json_data = json.dumps(data)
    return HttpResponse(json_data, content_type='application/json')
```

# The Whole Blueprint

Although I tried my best to explain details about this Django project, there are still nasty details you may want to consult from the code. So here is how the website looks like:

![image](https://raw.githubusercontent.com/sunshineatnoon/Django-Celery-Example/master/images/2.png)

This is the index page, which a user enters a number, and Celery calls a worker to start number times of Fourier Transforms.

![image](https://raw.githubusercontent.com/sunshineatnoon/Django-Celery-Example/master/images/3.png)

Here is what the progress looks like.

![image](https://raw.githubusercontent.com/sunshineatnoon/Django-Celery-Example/master/images/1.png)

And here shows the task is done.

# Code

The code is under MIT licence on my [GitHub](https://github.com/sunshineatnoon/Django-Celery-Example). A warning is that it doesn't work on Safari, but it works well on FireFox and Chrome.

# Reference

[1] [https://www.youtube.com/watch?v=Ip1OLq_-c2w](https://www.youtube.com/watch?v=Ip1OLq_-c2w)

[2] [https://github.com/NAThompson/learn_celery](https://github.com/NAThompson/learn_celery)

[3] [http://www.dangtrinh.com/2013/07/django-celery-display-progress-bar-of.html](http://www.dangtrinh.com/2013/07/django-celery-display-progress-bar-of.html)









