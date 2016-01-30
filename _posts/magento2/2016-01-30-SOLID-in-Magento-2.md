---
layout: post
title: Being a SOLID developer in Magento 2
categories: magento2
---

SOLID is a design pattern which developers should try to follow especially when making plugins for Magento 2.

Writing SOLID principles for Magento 2 is almost counter intuitive. If you're writing SOLID code properly, it shouldn't be coupled to Magento 2 at all. You're code should be for a particular purpose not a particular framework. 

By following these principles we will avoid code rot and we can de-couple from Magento all together; this means our innovative solutions will be available for not just the Magento 2 community but the whole of the PHP community. It also gives us the benefit of using other peoples libraries which is cool.

SOLID stands for:

- Single Responsibility
- Open-Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion 

##Single Responsibility

Single responsibility is probably the most known of the SOLID design principles. This means when writing a class it should only have one responsibility, or do one thing.

Common offenders of this rule are having validation or formatting methods within models or classes. We are all guilty of doing this, however doing so isn't very maintainable.

To write code which is legible and reusable you should write format or validation objects, their single responsibility would be to format or validate your other objects.

##Open-Closed

“software entities … should be open for extension, but closed for modification.” - [Wikipedia](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))

This is a very short and sweet explanation of the open-closed principle, however it might benefit from clearer definition.

To say "software entities … should be open for extension" means it should be easy to change the behaviour of a particular implementation. Whereas "closed for modification" means you shouldn't be editing or have to edit the original source code.

This doesn't just apply for other peoples code, you shouldn't touch other peoples code at all if you can help it; you should also think about this when writing your own code, especially when writing it in the first instance, you should be writing code that's easily extendible.

Example:

Currently I am creating an address finder from a postcode, the service we are using is Postcode Anywhere, however this might not always be the case. In the future we may want to implement Crafty Clicks, base functionality will be needed will be needed for both services.

If we were to switch to service to Crafty Clicks, I don't want to have to change the Postcode Anywhere source  code; we might want to switch this back, or it could be a config option whether the user wants to use Crafty Clicks or Postcode Anywhere.

I also wouldn't want to duplicate the logic for the shared functionality which is inevitably needed for the two classes, this means I will need to maintain two lots of code!

This may be a bit obvious to the more experience developer, however how we get around this is by having an abstract postcode object which the two services would extend. This would also mean we can add other services easily in the future if need be.

I would also use an interface, we will get onto that in the next few sections.

##Liskov Substitution

First off, it's called Liskov because it was introduced by Barbara Liskov.

This principle states that any new implementation should still adhere to all the same rules of its predecessor or class it extends. Basically, if you're modifying a current implementation by extension or dependency injection, we should not be breaking anywhere the functionality is already being used by keeping our method signatures consistent. 

This seems pretty obvious, however it can become more tricky than it sometimes seems. The trouble with using interfaces, we can control / validate the input, but not the output.

In PHP 7 I believe you can validate the output, but however lower versions of PHP are pretty flexible. Ways we can help validate outputs is by documenting our interfaces properly. This will help prevent changing the signature of methods.

##Interface Segregation

Interface segregation is a fancy way of saying you shouldn't include methods within your interfaces that the client may not use, doing this causes fat interfaces. 

Instead you should separate your contracts into separate interfaces, it is fine for a single class or implementation to implement multiple interfaces, in fact it's better.

By having multiple interfaces for one implementation or class, it means instead of passing a whole load of logic into a constructor or parameter we can pass just the interface that method or constructor needs. This makes adhering to the Liskov Substitution principle much easier.

{% highlight ruby %}
	interface VehicleInstructionInterface {

    public function startEngine();

    public function accelerate();

}

interface VehicleEngineInterface {
    
    public function checkOil(); 
    
}
{% endhighlight %}

{% highlight ruby %}
    Class Car implements VehicleInstructionInterface, VehicleEngineInterface  { 
        
    public function startEngine() {
        // broom     
    }  

    public function accelerate() { 
        // broooooom 
    } 

    public function checkOil() {
            // check oil     
    }
}
{% endhighlight %}

{% highlight ruby %}
    Class Driver {
    
    public function __construct(VehicleInstructionInterface $vehicle)
    {
        $vehicle->startEngine(); 
        $vehicle->accelerate();
    }
}
{% endhighlight %}


The driver doesn't need to know how to check the oil, this is a job for the mechanic. So even though a `Car` is being passed with the functionality, we don't need to know about the additional methods for our implementation to work.

You can see some great [examples here](http://code.tutsplus.com/tutorials/solid-part-3-liskov-substitution-interface-segregation-principles--net-36710) of this principle in action.

##Dependency Inversion

Dependency Inversion is not dependency injection, although they do go hand in hand an enables the dependency inversion principle.

Dependency inversion states that high level modules should not depend on low level modules. This allows us to decouple our code. High level code is  not concerned with a specific implementation, instead it should depend on a contract / interface. Low level code means the actual implementation logic such as models.

So going back to the example with the address finder earlier, the high level code would provide an interface to Magento or any other framework that they can hook into to get the address data. Where as the low level code would be the internals and would handle the nitty gritty like getting the code from Postcode Anywhere or Crafty Clicks.

The high level code should never change, this would break peoples implementation. However the low level code can be completely torn out a rewritten without breaking the implementation.

So the high level code is the abstraction, whereas low level code is the implementation. By coding to interfaces we will only rely on the high level code. 
