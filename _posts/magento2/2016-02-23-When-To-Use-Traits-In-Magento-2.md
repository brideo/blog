---
layout: post
title: When To Use Traits In Magento 2?
categories: magento2
---

The purpose of this post is actually somewhat inquisitive rather than informative, I will go over the basics of traits and provide and example. However it would be great if anybody from the community could point me in the direction of a well implemented `trait`, other than Fooman's which I mention below.

##What is a `trait`

A `trait` helps use with code reuse. You should think of it like copy and paste. If two implementations share the same functionality they may share methods. In this case you should almost never write those methods twice, you should either refactor your dependencies, think about inheritance or use a `trait`

A trait can own multiple methods and then you can use those traits in your classes.

##When to use a trait

Some months ago myself, [Vinai Kopp](https://twitter.com/vinaikopp), [Phil Winkle](https://twitter.com/philwinkle) and [Fooman](https://twitter.com/foomanNZ) were discussing when/if to use `traits` within your PHP code. You can see the Twitter discussion [here](https://twitter.com/brideoweb/status/671790263656935424).

Both sides to the discussion had interesting points, at the time I agreed with Vinai and couldn't really see the advantage of using a `trait`. I am still pretty undecided on the matter, I'm not sure whether this whole discussion just comes down to personal preference or I am missing something.

I have made an example of a `trait` in use below for people who haven't seen them before. Fooman also provided an [example with in Magento 2](https://github.com/magento/magento2/blob/develop/lib/internal/Magento/Framework/Interception/Interceptor.php) which he wrote for the platform, this is **a lot** better use case than mine.

##Example

Okay, so lets say we have to write two objects. The first one is a ticket office for a bus station, lets call is `BusStationTicketOffice()` and the second is a ticket office for an airport, lets call this one `AirportTicketOffice()`.
 
So both ticket offices need the ability to be able to create a passenger and set them into some kind of passenger repository, so first let's create a `PassengerHandlerInterface`.
 
{% highlight ruby %}
    <?php

 interface PassengerHandlerInterface
 {
 
     /**
      * Retrieve a passenger by name
      *
      * @return PassengerInterface
      */
     public function getPassenger($name);
 
     /**
      * @param PassengerInterface $passenger
      *
      * @return mixed
      */
     public function setPassenger(PassengerInterface $passenger);
 }
{% endhighlight %}

The problem is, the `BusStationTicketOffice` office and `AirportTicketOffice` are going to need the same logic when it comes to setting passengers into a repository or a protected variable. However they have already been tightly coupled to other implementations.
 
###How do we get around this?
 
I would almost always use dependency injection. IE rather than having my `BusStationTicketOffice` implement `PassengerHandlerInterface` I would inject some sort of `PassengerRepository` which implements the methods needed.
 
However in this example I am going to use a trait. As it is the `BusStationTicketOffice` and `AirportTicketOffice` job of creating customers from the data presented to them, we could do something like this:

{% highlight ruby %}
 trait TransportTrait
 {
 
     /**
      * @var array
      */
     protected $passengers = [];
 
     /**
      * @param array $passengers
      *
      * @return $this
      */
     public function setPassengers(array $passengers)
     {
         foreach ($passengers as $name) {
             $this->getPassengerFactory()->create(['name' => $name]);
         }
 
         return $this;
     }
 
     /**
      * Retrieve a passenger by name
      *
      * @return PassengerInterface|null
      */
     public function getPassenger($name)
     {
         if (!isset($passenger[$name])) {
             return null;
         }
 
         return $passenger[$name];
     }
 
     /**
      * @param PassengerInterface $passenger
      *
      * @return mixed
      */
     public function setPassenger(PassengerInterface $passenger)
     {
         $this->passengers[$passenger->getName()] = $passenger;
     }
 
     /**
      * @return PassengerFactory
      */
     abstract public function getPassengerFactory();
 }
 
{% endhighlight %}


{% highlight ruby %}
    class BusStationTicketOffice extends BusStationOffice implements PassengerHandlerInterface
 {
 
     use TransportTrait;
 
     /**
      * Bus constructor.
      *
      * @param PassengerFactory $passengerFactory
      * @param array            $passengers
      */
     public function __construct(
         PassengerFactory $passengerFactory,
         $passengers = []
     ) {
         $this->passengerFactory = $passengerFactory;
         $this->setPassengers($passengers);
     }
     
     /**
       * print all tickets
       *
       * @return $this
       */
     public function printTickets()
     {
        foreach($this->getPassengers() as $passenger) {
            $this->printTicket($passenger);
        }
     }
 
     /**
      * print the ticket for a passenger
      *
      * @return $this
      */
     public function printTicket($passenger)
     {
         $name = $passenger->getName();
         $address = $passenger->getAddress();
         
         // Some parent method
         $this->print($name, $address);
 
         return $this;
     }
 
     /**
      * Get the bus stations passenger factory.
      *
      * @return PassengerFactory
      */
     public function getPassengerFactory()
     {
         return $this->passengerFactory;
     }
 }
 {% endhighlight %}
 
 
 {% highlight ruby %}
    class AirportTicketOffice extends AirportTerminal implements PassengerHandlerInterface
 {
 
     use TransportTrait;
 
     /**
      * @var PassengerFactory
      */
     protected $passengerFactory;
 
     /**
      * Plane constructor.
      *
      * @param PassengerFactory $passengerFactory
      * @param array            $passengers
      */
     public function __construct(
         PassengerFactory $passengerFactory,
         $passengers = []
     ) {
         $this->passengerFactory = $passengerFactory;
         $this->setPassengers($passengers);
     }
 
       /**
       * check all passports
       *
       * @return $this
       */
     public function checkPassports()
     {
        foreach($this->getPassengers() as $passenger) {
            $this->checkPassport($passenger);
        }
     }
 
     /**
      * check the passport of a passenger
      *
      * @return $this
      */
     public function checkPassport($passenger)
     {
         //Some parent logic
         $this->scanInTerminal($passenger->getPassport());
         
         return $this;
     }
 
     /**
      * Get the airports passenger factory.
      *
      * @return PassengerFactory
      */
     public function getPassengerFactory()
     {
         return $this->passengerFactory;
     }
 }
{% endhighlight %}

##Conclusion

This is actually a pretty bad example of a use case for a `trait`, I would almost certainly us DI here.

I suggest taking a look at Fooman's example, it is the best I have seen of a `trait`.

I would really like to see some more real life examples of traits, Kalen Jordan suggested using a `trait` for a logger which I could see as being useful.

A good resource regarding traits is [Cullt's Blog](http://culttt.com/2014/06/25/php-traits/).
