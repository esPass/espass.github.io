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

We have 2 fields to provide timing information. One is to define times when the ticket is valid ( "valid_timespans") - the other ("calendar_timespan") is to give the user a nice calendar-entry.
If the valid_timespans field is empty: the tickets are always valid. But there are for example use-case like annual passes where you want to define a start and a end-date. In some cases even multiple of those - hence this is defined as an array. Repetitions are also supported - e.g. for bus-tickets that are only valid on Fridays. The offset is given in hours.
From and to are also optional - if "from" is left out then it defaults to the beginning of time and if "to" is left out - then it defaults to the end of time. This is useful if the pass is imported from pkpass format where we only have the "expirationDate" field.


{% highlight json %}
{
  "validTimespans": [{
    "from":"2016-12-28T19:00+01:00",
    "to":"2016-12-28T23:00+01:00",
    "repeat": {
      "offset":168,
      "count":52
    }
  }]
}
{% endhighlight %}

  If no calendar_timespan is given the app will not present the user with the functionality of easily adding a calendar entry that fits the pass. For some use-cases like some annual pass  this would not be a useful feature. For other cases like some concert - this can be a pretty nice time saver. Repeat will be of no use for the calendar - but still the same data structure can be used as repeat is optional.

{% highlight json %}
{  
  "calendarTimespan": {
    "from":"2016-12-28T20:00+01:00",
    "to":"2016-12-28T22:00+01:00"
  }
}
{% endhighlight %}


From and to are optional - but one of them has to be present - if "from" is not given it defaults to beginning of time - and "to" defaults to end of time. Passes that origin on the pkpass format will only have to as we only have the expirationDate field here.
All time information in esPass has to be in the ISO-8601 format.

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
    "name":"CCH Hamburg",
    "lat": 53.561826,
    "lon": 9.986032
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

### Barcode

With the barcode section we can represent passes as used currently ( 2016 ) in this format. This might also be used in the future when no blockchain connection is needed ( e.g. for passes to events where no reselling is wanted and the recipients are known)

{% highlight json %}
{
  "barCode": {
    "format": "QR_CODE",
    "message": "13f3c625-ec9e-40cf-b8eb-85eedb765cf9",
    "alternativeText": "13f3c625-ec9e",
    "size":"128x128"
  }
}

{% endhighlight %}

Format can be either QR_CODE, PDF417, AZTEC, CODE_39, CODE_93, CODE_128
The message is the message that should be encoded with the barcode.
alternativeText and size are optional. The alternativeText is displayed below the barcode if given and used for manual verification if there are problems with the barcode reader. Size gives a indication how big the barcode should ideally be to be optimally scanned by the scanners available at the entrance. The size is given in [Device independent pixels](https://en.wikipedia.org/wiki/Device_independent_pixel)

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


Processes
---------

### Reselling

If you sell a pass - you do not just give a copy of the esPass file to the buyer. This would defeat the purpose. Reselling has to be done over an app that calls a contract to reassign the pass from the seller to the buyer. The buyer never discloses his private key for the pass - and only this way he can check and be sure later on that only he gets access - because only he can prove to have the private key which is assigned with this pass.

### Entry

When checking passes at the entrance we check that the guests have the private key assigned with the pass. We can do this by letting the user sign a given string with their key. We might define a freeze-period before the event where tickets cannot be resold anymore as we otherwise have to rely on a on the spot internet connection.

### Graceful degradation

Not everyone has a smart phone - but we can degrade gracefully for them. When delivering the pass we can deliver the signed message for the initial key in a barcode as before. This way you cannot resell but you can gain entrance. You have to keep the pass secret as before - nothing changes there. This barcode can e.g. be delivered as an image alongside the espass file in the email after buying and users can print it.

### Key storage

We store the key in the esPass file. There might also be the possibility to use keys from some wallet-app later on but to get a really low barrier and pleasant user experience in the beginning we go this route.

### Who pays the ether/gas

Also for reasons of good UX we will not require the normal user to hold any ether. The ether needed will be provided by the creator of the passes he pays the fee when instantiating the contract - he will price this in when selling the passes. It should be fraction of cents per pass. When reselling the pass there is also ether needed - but then there is mostly also money involved and we can use a fraction of this to buy ether to execute the transfer function of the contract.

### Distribution of passes

In the beginning we will be pragmatic about this and deliver passes via download or mail. But the future is in providing the passes via [IPFS](https://en.wikipedia.org/wiki/InterPlanetary_File_System). This will only work for contract based passes. It would be stupid with barcode backed passes. Contract based passes will be provided without the key information. So the pass provider can publish the pass with all needed information - the user can download this and then make the pass a valid one by paying the pass and executing a contract this way. The hash in ipfs can then also be bound to the event-contract so users can be sure about details e.g. specified in the fields.
