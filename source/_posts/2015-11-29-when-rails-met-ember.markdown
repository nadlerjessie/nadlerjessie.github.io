---
layout: post
title: "When Rails Met Ember"
date: 2015-11-29 16:03:30 -0500
comments: true
categories: "API Ember Flatiron Rails"
---

During the past week and a half at the Flatiron School I have been learning about Ember. The big brewing question in my mind while learning different features was, how does this work with Rails? I worked on labs where the back-end was done for me and I only had to build out the front-end to make requests to the Rails API. I know this is something we will cover next week but I was too eager to wait. Can you think of a better way to spend the day before Thanksgiving?!

The purpose of this blog post is to go through the basic steps of creating an App with a Rails front-end and an Ember back-end. This post assumes you know how to build apps in both Rails and Ember but need more information on how to connect them.

**Create a Rails app**  <br>
That's easy enough! There are two ways to do this. I used rails new and later added the rails-api gem to my gemfile. Alternatively, you can install the gem in your terminal ```gem install rails-api``` and generate a new Rails::API with ```rails-api new my_api```
This gem makes ApplicationController inherit from ActionController::API instead of ActionController::Base. If you generate your app the standard way, this is a change you will have to make manually to your app/controllers/application_controller.rb file. Be sure to comment out the protect_from_forgery line as well. For extra information on this gem, refer to the <a href= "https://github.com/rails-api/rails-api">README on github</a>. <br>

**Add gems**  <br>
In additon to the rails-api gem, there are a few others needed to correctly configure your app as an API. The first is rack-cors. This is a piece of Rack Middleware. It will make which makes cross-origin AJAX possible by handling Cross-Origin Resource Sharing (CORS). This is critical because otherwise your Ember app won't be able to make requests to your Rails app! Add ```gem 'rack-cors', :require => 'rack/cors'``` to your gemfile. In config/application.rb add the following inside YourApp::Application 
```
config.middleware.insert_before 0, "Rack::Cors" do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end
```
The '*' for origins is a wildcard that allows for requests to be made from any origin. If you know specifically which origin(s) you want to allow access from, maybe your localhost port or an existing web app, replace the star with the accepted URL(s). Refer to the <a href= "https://github.com/cyu/rack-cors">README on github</a> for more on rack-cors.

The other gem to include is active_model_serializers. This will allow you to serialize your ActiveModel/ActiveRecord objects into your desired format (ex: JSON). Include ```ActionController::Serialization``` in your application controller. <br>

**Define associations**  <br>
Set up your schema and define your associations as you normally would in the models. There is also a folder for serialzers, which needs to be configured. You will need to specify which attributes and associations you want to include in the serialzed form. For example:
```
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :body
  has_many :comments
end
```
Having the full comment may not be ideal for your application. This will depend on how many associations you have and the size of your models. Adding ```embed :ids, include: true``` will embed the ids of the comments in the JSON response. For more, look at the <a href= "https://github.com/rails-api/active_model_serializers/tree/0-9-stable">README on github</a>. <br>

**Create an Ember app**  <br>
Run ```ember new my-app```. Set up your models to match the responses from the Rails side. For the example above, the app/models/post.js file would look like:
```
import DS from 'ember-data';

export default DS.Model.extend({
  title: DS.attr('string'),
  body: DS.attr('text'),
  comments: DS.hasMany('comments', {async: true})
});
```
Next you will need to install active-model-adapter. This will act as both an adapter and a serializer. Run ```ember install active-model-adapter```. If you don't already have an application adapter, generate one. It should be in app/adapters/application.js. Replace the content with the following:
```
import ActiveModelAdapter from 'active-model-adapter';

export default ActiveModelAdapter.extend({
  namespace: "api/v1",
  host: "http://localhost:3000"
});
```
The namespace should reflect any namespacing done in the Rails side. Though not explicitly stated above, it is convention to have the controllers to be in a namespace of API and the version number. <br>

**Finish up the front-end**<br>
Define your models, add any components you may need, create your templates and do whatever else you need the front-end of your application to have! 
