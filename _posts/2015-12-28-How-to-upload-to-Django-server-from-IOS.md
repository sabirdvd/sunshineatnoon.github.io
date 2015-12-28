---
layout: post
title: How to upload image from IOS to Django server
published: true
---

I have been bumping my head to the wall for days to implement this function. From GitHub to StackOverFlow, finally, it worked like a charm. So I decided to keep a note in case it might be helpful to others.
## Install AFNetwork
[AFNetwork](https://github.com/AFNetworking/AFNetworking) is a popular networking library for iOS and Mac OS X. With the help of this framework, it is easy to send a POST request from our IOS app to any web server, including a Django server. Installing is easy, just follow the official steps, here I make a summary:

- Install cocoapods: `gem install cocoapods`
- Setup cocoapods: `pod setup`
- Create a new Xcode project and go to the folder where your YourProjectName.xcodeproj file is, then create a Podfile:  `touch Podfile` 
- Add these lines to the file, I am currently working with IOS 9 and AFNetowrk 2.5, feel free to change it to any IOS version you are working with: 

  ```
  source 'https://github.com/CocoaPods/Specs.git'
  platform :ios, '9.0'
  pod 'AFNetworking', '~> 2.5'
  ```
- Install the dependencies in your project, btw, if this step takes like forever on your computer, you might want to refer to [this answer](http://stackoverflow.com/questions/23755974/cocoapods-pod-install-takes-forever) on StackOverFlow: `pod install`
- The last tip is important, since from now on, you don't open your Xcode project by double click or by Xcode directly because you get a workspace now, so open your project by this command:`open YourProjectName.xcworkspace`


## Implement IOS Side
First, we need to import AFNetwork in order to use its functions:

`#import <AFNetworking/AFNetworking.h>`

Then, we POST an image to the server when a button is clicked:

```
- (IBAction)BtnClick:(id)sender {

    AFHTTPRequestOperationManager *manager = [[AFHTTPRequestOperationManager alloc] initWithBaseURL:[NSURL URLWithString:@"http://server.url"]];  
    UIImage *image = [UIImage imageNamed:@"dog.jpg"];
    NSData *imageData = UIImageJPEGRepresentation(image, 0.5);
    NSDictionary *parameters = @{@"name":@"sunshineatnoon"};
    AFHTTPRequestOperation *op = [manager POST:@"http://your.ip.address:8000/myapp/list/" parameters:parameters constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
    [formData appendPartWithFileData:imageData name:@"docfile" fileName:@"photo.jpg" mimeType:@"image/jpg"];
    } success:^(AFHTTPRequestOperation *operation, id responseObject) {
        NSLog(@"Success: %@ ***** %@", operation.responseString, responseObject[@"top_2"]);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"Error: %@ ***** %@", operation.responseString, error);
    }];
    op.responseSerializer.acceptableContentTypes = [op.responseSerializer.acceptableContentTypes setByAddingObject:@"text/html"];
    [op start];    
}

```
I want to make few notes about the objective-c code above:

- `dog.jpg` is the image we upload from our app to the server
- `http://your.ip.address:8000/myapp/list/` is the url of our Django server, you might want to open link from your browser to make sure you don't get a 404 first
- you need to double check that the paramters and file name your specify matches what you define in the server side, other wise the form we post to the server will be invalid. For instance, my DocumentForm in the server side is defined like below, thus I have `@{@"name":@"sunshineatnoon"}` in the paramters and named my imageData to be docfile: `[formData appendPartWithFileData:imageData name:@"docfile" fileName:@"photo.jpg" mimeType:@"image/jpg"];`

```
from django import formsclass DocumentForm(forms.Form):    #docfile = forms.FileField()    name = forms.CharField(max_length=100)    docfile = forms.FileField()
```

   
## Implement the Django Server

[Django](https://www.djangoproject.com) is a great python web framework. My Django server largely based on this [GitHub repo](https://github.com/axelpale/minimal-django-file-upload-example) since it already implements a web demo of uploading image to a Django Server. Here I work with Django 1.8.

######First we need to change the form so we can upload a string and an image to the server, change `forms.py` to this:

```
from django import formsclass DocumentForm(forms.Form):    #docfile = forms.FileField()    name = forms.CharField(max_length=100)    docfile = forms.FileField()
```
Then we need to define what we want to return to our IOS App, for now, I will just return 1 if Django server receives valid form or 0 if the form is invalid. So our `views.py` is like this:

```from myproject.myapp.models import Documentfrom myproject.myapp.forms import DocumentFormfrom django.http import HttpResponseimport jsonfrom django.views.decorators.csrf import csrf_exempt@csrf_exemptdef list(request):    # Handle file upload    if request.method == 'POST':        form = DocumentForm(request.POST, request.FILES)        if form.is_valid():            newdoc = Document(docfile=request.FILES['docfile'])            newdoc.save()            # Redirect to the document list after POST            return HttpResponse(json.dumps({"Status": 0}, sort_keys=True))        else:            return HttpResponse(json.dumps({"Status": 1}, sort_keys=True))
```
A special attension should be paid to the csrf, since this is only a pretty simple demo, I just disable csrf, otherwise the server will decline to accept the POST request unless it provides a valid csrf.

## Result
Now let's set up our server:

```
$: cd minimal-django-file-upload-example/src/for_django_1-8/myproject/
$: python manage.py migrate
$: python manage.py runserver 0.0.0.0:8000
```
Now if you can visit your server at `http://0.0.0.0:8000/myapp/list/` locally or `http://your.server's.ip.address:8000/myapp/list/` from another computer. Of course you will get the error `The view myproject.myapp.views.list didn't return an HttpResponse object. It returned None instead.`. This is because we only return a number instead of a html file, but it at least tells us that our site is recheable.

What about our IOS APP? Let's try. Run the app and click the upload button, then we see this output from the log:
```
2015-12-28 21:30:21.648 uploadTest[6090:210046] Success: {"Status": 0} ***** 0
```
This means we have successfully send our POST request to the server. So does our server successfully receive the image? We can check this path `minimal-django-file-upload-example/src/for_django_1-8/myproject/media/documents/2015/12/28`. See, let's our image!


Should you encounter this problem:`Transport Security policy requires the use of a secure connection.`, see [here](http://stackoverflow.com/questions/32631184/the-resource-could-not-be-loaded-because-the-app-transport-security-policy-requi) for a solution.

## Reading image from the server
To send a string from the Django server back to IOS APP is easy, just wrap the string in the json, as I did to return the Status. But how about an image? Well, that's a little more complicated. A trikcy way is to read that image directly from the server by its url. For instance, the image we just upload can be viewed by this url: [http://your.server.ip.address:8000/media/documents/2015/12/28/photo.jpg](http://your.server.ip.address:8000/media/documents/2015/12/28/photo.jpg), So we can directly read the image back by this url. The code below implements this function:

```
NSString *urlStr = [NSString stringWithFormat:@"http://your.server.ip.address:8000%@",responseObject[@"ImgName"]];
        NSURL *url = [NSURL URLWithString:urlStr];
        
NSData *data = [NSData dataWithContentsOfURL:url];
UIImage *img = [UIImage imageWithData:data];
        
self.imgViewController.image = img;
```

## Code 
I uploaded all the code to my [GitHub]().

## Reference

[1] [https://github.com/axelpale/minimal-django-file-upload-example](https://github.com/axelpale/minimal-django-file-upload-example)

