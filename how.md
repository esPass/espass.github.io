---
title: How does esPass work
layout: default
permalink: /how/
---

Foreword
--------


### BlockChain provider

The first iteration will use the [Ethereum](http://ethereum.org) BlockChain exclusively. Smart contracts are key to smart passes - hence the similarity in naming. But esPass will be open to allow other BlockChains in the future.


### A word on timing

There is an important piece missing in Ethereum for the intended use-case currently. As smartPasses will be mobile-centric we rely on [light-client](https://github.com/ethereum/wiki/wiki/Light-client-protocol) support - this is planned for Ethereum but not yet available. Get information about the current state of light-clients on Ethereum [here](gitter.im/ethereum/light-client). But this does not have to block us to prepare this standard. This way we can directly use it when light-client support becomes available.

Contracts
---------

### simple event-passes

The basic smartPass is like the [coin contract](https://www.ethereum.org/token) - just instead of coins we manage tickets with this contract.

{% highlight java %}
contract MyEventPass {         
  /* map from address to count of tickets */
  mapping (address => uint256) public ticketOwnership;   

  function MyEventPass() {
    ticketOwnership[msg.sender] = 1000; /* 1000 Tickets available */
  }

  /* Send passes */
  function transfer(address _to, uint256 _value) {
      if (ticketOwnership[msg.sender] < _value) throw;           // Check if the sender has enough   
      if (ticketOwnership[_to] + _value < ticketOwnership[_to]) throw; // Check for overflows
      ticketOwnership[msg.sender] -= _value;                     // Subtract from the sender
      ticketOwnership[_to] += _value;                            // Add the same to the recipient            
  }

  /* This unnamed function is called whenever someone tries to send ether to it */
  function () {
      throw;     // Prevents accidental sending of ether
  }  
}
{% endhighlight %}

the esPass FileFormat
---------------------

### container

The pass is a zip-file with the file-extension .espass - In this zip-container you can find a main.json file that mainly defines the pass. You also find assets like icons in this container.

### mandatory fields

{% highlight json %}
{
  "type":"event",
  "version":"1",
  "name":"My awesome event"
}
{% endhighlight %}

type can be either *event*, *voucher*, *loyalty*, *coupon*, *boarding*

this might be extended with other use cases in later versions

we keep the mandatory fields as limited as possible to not pollute later usages with unnecessary fields.

### time info

if no valid_time_ranges field is present - the tickets are always valid. But there are for example use-case like annual passes where you want to define a start and a end-date. In some cases even multiple of those - thats why we give flexibility with an array here.

{% highlight json %}
{
  .. other fields ..
  "valid_time_ranges": [{
    "from":"",
    "to":""
  }]
}
{% endhighlight %}


### location info

The location info serves 2 main purposes - for one we can give the user some easy navigation to the event from the pass - for the other you can present the user easy access to the pass ( e.g. via some notification ) when he gets close to the location ( at the right time ).

{% highlight json %}
{
  .. other fields ..
  "locations": [{
    "name","",
    "lat":"",
    "lon":""
  }]
}

{% endhighlight %}

every field here is optional - this way you can give one event just one location-name - or just add latitude and longitude. There can be multiple locations - think about the usage for ski passes - and the different locations are different lifts.
If you specify latitude or longitude - then the counterpart must be given - they shall never stand alone. Later we will also add beacon information to this section.


### design

In the design section you find only color definitions for the foreground and background color for now. Colors must be ARGB prefixed with #

{% highlight json %}
{
  .. other fields ..
  "design": {
    "fgColor":"#FFFF0000",
    "bgColor":"#FF00FF00"
  }
}

{% endhighlight %}


### contract info

It is not mandatory for a SmartPass to be smart and be backed by the BlockChain - even without this we will have some advantages to the Passbook format ( e.g. time-spans ). For the first period and use-cases that need no backing by BlockChain - this field can be just left out.

{% highlight json %}
{
  .. other fields ..
  "contract": {
    "chain":"eth",
    "contract":"0x4242424242",
    "key":"0x232323232323",
    "interface","simple_event"  
  }
}
{% endhighlight %}

chain specifies the chain - as mentioned in the beginning we will be BlockChain agnostic and support other BlockChains in the future. Our only option here in the beginning is eth for Ethereum - which does not mean the Ethereum technology - but the Ethereum ( main ) BlockChain - there might be other BlockChains that use Ethereum technology
