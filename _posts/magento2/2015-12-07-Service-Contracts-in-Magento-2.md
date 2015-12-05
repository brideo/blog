---
layout: post
title: Service Contracts Magento 2 (Part 1)
categories: magento2
---

The main and most exciting change in Magento 2 is that service contracts have been implemented by using [interfaces](http://php.net/manual/en/language.oop5.interfaces.php).

##What are PHP `interfaces`?

Interfaces provide a set of rules or a service contract for a class telling the developer that it should contain a certain set of methods or properties.  

You should have a class implement and interface if in the future the implementation could change. A good example of this would be a `SearchInterface`.

This example is not specific to Magento, lets say you're implementing Elastic Search on your website. You might start by creating an `ElasticSearch` class with a few rudimentary methods:

{% highlight ruby %}
    <?php
    
    class ElasticSearch {
    
        public function search()
        {
        // Search the index
        }
        
        public function get($id)
        {
        // Get an object from index
        }
        
    }
{% endhighlight %}

This is all well for now, but what if you now want to implement a new search engine? You will have to write a whole. Writing a whole new `class` could break the current website or any API implementations you may have.

Rewriting a new `class` is going to be a massive pain, you'll going to have to reference the original and then check the rest of your app to check it hasn't broken anything else.

Instead of this we can use an `interface`. Remember, an `interface` is a service contract which promises that a `class` will implement certain methods when implementing it.

 Here is what our interface would look like:

{% highlight ruby %}
    interface SearchInterface {
        
    public function search();
    
    public function get($id);
    
}
{% endhighlight %}
   
The client wants to now migrate to `sphinx`, now we have a service contract in place we know when we implement this interface we must use these certain methods:

{% highlight ruby %}
    class Sphinx implements SearchInterface {
        
        public function search()
        {
            // New search logic
        }
        
        public function get($id)
        {
            // New get logic
        }
    }
{% endhighlight %}

##Service Contracts in Magento 2

Service contracts in Magento 2 are extremely powerful. Magento can tell us exactly what methods it needs to implement a certain feature. This means we are no longer so tied down to the framework, we can decouple our modules and make full use of the libraries within the PHP community. 

Magento modules now only communicate through an API, the service contract will never change however the implementation internals can be completely revamped as long as we keep the same interface.

This means we can concentrate on using, improving and writing the best PHP libraries rather than specific Magento modules. 
 
 There are two types of API in Magento 2:
 
 - The Data API
    - Provides access to CRUD operations that belong to modules
    - Found in `module_name/Api/Data`
 - The Operational API
    - Provides access to data and drives operations used
    - Should provide access your public methods that would of typically been in `Helpers` and `Models` in Magento 1.
    - Found in `module_name/Api/`
 
 The Data API only exposes the CRUD methods, whereas the operational API actually does the work.
 
###Whats Next?
 
 I will be covering the Magento Framework API implementation in my next post, however it's a pretty big section so I am going to split this topic into 2 or maybe 3 sections. 
