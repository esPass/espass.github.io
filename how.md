---
title: How does esPass work
layout: default
permalink: /how/
---

Context
-------

### Blockchain

The first iteration will use the [Ethereum](http://ethereum.org) BlockChain exclusively. Smart contracts are key to smart passes - hence the similarity in naming. But esPass will be open to allow other BlockChains in the future.


### Timing

There is an important piece missing in Ethereum for the intended use-case currently. As smartPasses will be mobile-centric we rely on [light-client](https://github.com/ethereum/wiki/wiki/Light-client-protocol) support - this is planned for Ethereum but not yet available. Get information about the current state of light-clients on Ethereum [here](https://gitter.im/ethereum/light-client). But this does not have to block us to prepare this standard. This way we can directly use it when light-client support becomes available.

### Status

This is a draft. Please join the discussion - feedback and pull-requests are very welcome. Please expect changes and don't rely on this to be backward compatible yet.

Contracts
---------

### Simple event-passes

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

The esPass FileFormat
---------------------

### Container

The pass is a zip-file with the file-extension .espass - In this zip-container you can find a main.json file that mainly defines the pass. You also find assets like icons in this container.

### Mandatory fields

{% highlight json %}
{
  "type":"EVENT",
  "name":"My awesome event",
  "id":"db8049ec-f576-4548-aee8-c6b5f5004909"
}
{% endhighlight %}

type can be either *EVENT*, *VOUCHER*, *LOYALTY*, *COUPON*, *BOARDING*

this might be extended with other types in later versions

"Id" is mainly used to prevent duplicates and is a string that should be as unique as possible.

Name is a string that can be presented to the user in a list of passes.

We keep the mandatory fields as limited as possible to not pollute later usages with unnecessary fields.

### Time info

If no valid_time_ranges field is present: the tickets are always valid. But there are for example use-case like annual passes where you want to define a start and a end-date. In some cases even multiple of those - thats why we give flexibility with an array here.

{% highlight json %}
{
  "valid_time_ranges": [{
    "from":"2016-12-28T20:00+01:00",
    "to":"2016-12-28T23:00+01:00"
  }]
}
{% endhighlight %}

### Metadata

{% highlight json %}
{
  "app":"passandroid",
  "creator":"ligi"
}
{% endhighlight %}

The app field indicates which application was used to create the pass. This can enable special treatment for passes from certain sources. The creator field states who created the pass. This info can be used for grouping passes. It can also be used to give the user a clue if a new pass is from the same vendor as before. This will be combined with a check if the key which was used for signing is the same.

### Location info

The location info serves 2 main purposes - for one we can give the user some easy navigation to the event from the pass - for the other you can present the user easy access to the pass ( e.g. via some notification ) when he gets close to the location ( at the right time ).

{% highlight json %}
{
  "locations": [{
    "name":"",
    "lat":"",
    "lon":""
  }]
}

{% endhighlight %}

every field here is optional - this way you can give one event just one location-name - or just add latitude and longitude. There can be multiple locations - think about the usage for ski passes - and the different locations are different lifts.
If you specify latitude or longitude - then the counterpart must be given - they shall never stand alone. Later we will also add beacon information to this section.

### Fields

Fields is a list of fields with information to the user. This might be e.g. which entrance to take - or which seat you have if seats are assigned. There are a lot of possibilities for fields and this highly depends on the type of event. This list might also be empty.

{% highlight json %}
{
  "fields":[{
    "label":"Film",
    "value":"Star Trek - Beyond",
    "hide":false
  }]
}
{% endhighlight %}

Fields always consist of a label and value pair - the hide flag gives a hint to the presenter that this field should not be shown by default - only after user action - might be something like Terms of Service information - similar to the backFields in the pkpass format

### Color

The accent color can give a visual clue to faster find passes and give some pleasure to the eyes.

{% highlight json %}
{
  "accentColor":"#FFFF0000"
}

{% endhighlight %}


### Contract info

It is not mandatory for a SmartPass to be smart and be backed by the BlockChain - even without this we will have some advantages to the Passbook format ( e.g. time-spans ). For the first period and use-cases that need no backing by BlockChain - this field can be just left out.

{% highlight json %}
{
  "contract": {
    "chain":"eth",
    "contract":"0x4242424242",
    "key":"0x232323232323",
    "interface":"simple_event"
  }
}
{% endhighlight %}

chain specifies the chain - as mentioned in the beginning we will be BlockChain agnostic and support other BlockChains in the future. Our only option here in the beginning is eth for Ethereum - which does not mean the Ethereum technology - but the Ethereum ( main ) BlockChain - there might be other BlockChains that use Ethereum technology
