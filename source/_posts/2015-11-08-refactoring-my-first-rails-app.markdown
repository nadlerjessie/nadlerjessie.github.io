---
layout: post
title: "Refactoring My First Rails App"
date: 2015-11-08 16:41:39 -0500
comments: true
categories: 
---

Last week I worked in a group to create my first Rails app! I had only created one other app from scratch in the past, which was done in Sinatra. A group of fellow Flatiron students and I made a dinner party organizer, currently titled <a href = "https://github.com/nadlerjessie/dinner_party/tree/master/DinnerParty">Bring It</a>. I was really excited by the project and was ready to get some of the functionality going. Unfortunately, this caused me to write some code that wasn't very DRY. 

This project gave me a new appreciation for the refactoring process, which I hadn't yet done on such a big scale. In this post I'll take you through the cycle of just one section of code. 

**The Problem:**<br>
I have a class called MenuItems, which belongs to a Dinner as well as a Dish. This means that each menu item would be for a specific dinner party as well as a specific dish with information about the food itself. However each dish also had a type of course: main dishes, salads, appetizers and desserts. On the views I wanted to group the dishes by their course so they could have appropriate headers, like a normal menu.

**Round 1: My Initial Code [class MenuItemsController]**<br>
In my controller I wrote the following code:
```ruby
def index
  @main_dishes = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Main Dish"})
  @salads = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Salad"})
  @appetizers = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Appetizer"})
  @desserts = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Dessert"})
end
```
There are clearly a ton of issues with this code. A short list of issues:

1) When passing these instance variables to the views, I needed to have different HTML calls to iterate through each one<br>
2) It's incredibly repetitive<br>
3) The database searches were happening in the controller<br>
4) The courses are hardcoded

But it worked and that was what I needed to get started! Red, green, refactor... right? The first bit of refactoring I did was to alleviate problem number 1 stated above. I wanted to remove the redundancy from the view.

**Round 2: A First Attempt to Refactor [class MenuItemsController]**
```ruby
def index
  main_dishes = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Main Dish"})
  salads = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Salad"})
  appetizers = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Appetizer"})
  desserts = MenuItem.joins(:dish).where(dinner_id: dinner_id, dishes: {course: "Dessert"})
  @all_dishes = [main_dishes, salads, appetizers, desserts]
end
```

This was really no better on the controller side but it allowed me to iterate through @all_dishes in my index view rather than having separate code to display each course. At this point I wisened up and decided I didn't want four separate calls and I didn't want any of the database searches to be in the controller. In my MenuItem model I wrote the following:

**Round 3: Moving Code to the Model**<br>
In the MenuItem model I wrote:
```ruby
@@courses = ["Main Dish, Salad, Appetizer, Dessert"]

def self.index_by_course(dinner_id)
  @@courses.map do |course| 
    self.joins(:dish).where(dinner_id: dinner_id, dishes: {course: course})
  end
end
```
And in the MenuItemsController:
```ruby
def index
  @menu_items_by_course = MenuItem.index_by_course(@dinner.id)
end
```
This was definitely feeling like a step in the right direction. One remaining issue was that I was defining courses as a class variable in MenuItem even though it was a property of Dish. This got me into some trouble later. I had written similar code to organize all of the dishes by their course (as opposed to just dishes for this specific dinner menu, or menu items) and refactored similarly as well. All of a sudden on one of my pages I didn't see the main dishes! I was already weary of my definition of courses, especially because I had it in two places. It turned out, I had typed one as "Main Dishes" and another as "Main Dish". It was definitely time to remove this redundancy. 

**Round 4: Adding a CONSTANT!**<br>
Course was defined as a property of a dish, so I wanted to add the constant there. The dish model became:
```ruby
class Dish < ActiveRecord::Base
  has_many :menu_items
  has_many :dish_assignments, through: :menu_items

  COURSES = ["Main Dishes", "Sides", "Appetizers", "Desserts", "Beverages"]

  def self.find_available_dishes(dinner_id, menu_ids)
    COURSES.map do |course|
      Dish.select("dishes.*").where(course: course).where.not(id: menu_ids)
    end
  end  
end
```
I was then able to update my menu_items_by_course class method to:
```ruby
def self.index_by_course(dinner_id)
  Dish::Courses.map do |course| 
    self.joins(:dish).where(dinner_id: dinner_id, dishes: {course: course})
  end
end
```

**Reviewing the Issues**<br>
Going back to my short list of issues with my initial code, I can see how those issues were resolved.

1) *When passing these instance variables to the views, I needed to have different HTML calls to iterate through each one* - **I solved this one early on in my refactoring process. First by passing four different database calls into an array before passing the instance variable to the view and later by mapping results while iterating over courses.**<br>
2) *It's incredibly repetitive* - **Much of this repetitive nature has been reduced using iteration**<br>
3) *The database searches were happening in the controller* - **Database searches were now moved to the model**<br>
4) *The courses are hardcoded* - **While the courses are listed in the dish model, they are only listed in one place. This makes debugging easier and gives much more flexibility! I ended up changing the course options to Main Dishes, Sides, Appetizers, Desserts, and I added Beverages, which was all easy because they were defined in a single place. As a result of making my code more DRY I didn't have to made modifications in the other models, controllers, or views!**

I know that I will always be able to refactor code after I write it but hopefully by reflecting on my refactoring process I will write cleaner code in earlier stages. 









