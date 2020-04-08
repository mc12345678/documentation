---
title: Notifiers and Observers - About
description: About Zen Cart Notifiers and Observers 
category: code
weight: 10
---

### Introduction

One of the many goals of the Zen Cart project has always been to make it easier for third party developers to add functionality to the core code in an easy and unobtrusive manner. To do this we have in the past relied on the override and auto inclusion systems. However these still do not give developers an easy method of hooking into many areas of core code, without 'hacking' core files themselves.

The observer/notifier system was introduced to give developers unprecedented access to core code, without the need to touch any core files at all. Although ostensibly written for an object-oriented code base, we will see later how it can be used with general procedural code as well.

### Extending All Classes

In order to implement the observer/notifier system, some structural changes have been made to Zen Cart. Firstly two new classes have been introduced: the base class (`class.base.php`) and the notifier class (`class.notifier.php`).

The base class contains the code that is used to implement the observer/notifier system. However to make it effective all other Zen Cart classes now have to be declared as children of the base class. You will see this if you look at the source of any of the Zen Cart classes.

<pre>  class currencies extends base {
</pre>

The notifier class will be discussed later, when we look at extending the observer/notifier system (ONS) into procedural code.

### Notifiers: Big Brother is watching

So, what is all the fuss about?

The point of the ONS is that developers can write code that wait for certain events to happen, and then when they do, have their own code executed.

So, how are events defined, where are they triggered?

Events are triggered by code added to the core for v1.3 (with more to be added over time). In any class that wants to notify of an event happening we have added:

<pre> $this->notify('EVENT_NAME');
</pre>

An example would probably help here:

In the shopping cart class after an item has been added to the cart this event is triggered:

<pre> $this->notify('NOTIFIER_CART_ADD_CART_END');
</pre>

There are many other events that have notifiers in Zen Cart v1.3 and newer; for a list [see here](/dev/code_notifier_list).

All of this notifying is all well and good, but how does this help developers?

### Observe and Prosper

To take advantage of notifiers, developers need to write some code to watch for them. Observers need to be written as a class. There's even a nice directory, `includes/classes/observers`, where developers can put these classes.

Lets take an example. Using the notifier mentioned above (`NOTIFIER_CART_ADD_CART_END`), how would I write a class that watched for that event?

```
<?php
 class myObserver extends base {
   function __construct() {
     $this->attach($this, array('NOTIFIER_CART_ADD_CART_END'));
   }
...
 }
```

As you can see we have defined a new class called myObserver and in the constructor function for that class (function myObserver) have attached this myObserver class to the event `NOTIFIER_CART_ADD_CART_END`.

"Fine," I hear you saying, "but how do I actually do anything useful?"

Ok, good question. Whenever an event occurs, the base class looks to see if any other observer class is watching that event. If it is, the base class executes a method in that observer class. Remember the `$this->notify('EVENT_NAME')` from above? Well, when that event occurs, the base class calls the update method of all observers. Lets see some more code:

```
class myObserver extends base {
   function __construct() {
     $this->attach($this, array('NOTIFIER_CART_ADD_CART_END'));
   }
   function update(&$callingClass, $notifier, $paramsArray) {
     ... do some stuff
   }
 }
```

Now, whenever the `NOTIFIER_CART_ADD_CART_END` occurs, our `myObserver::update` method will be executed. Note that `attach()` may be called as a method of whatever class you want to listen to (`$_SESSION['cart']`, in this case) or by the internal class variable $this. Both are available since each are part of the class base, where the attach method resides.

Some notes about the parameters...the attach method has two parameters:

*   &$observer - Reference to the observer class, used to generated a unique ID for the new listener
*   $eventIDArray - An array of notifiers that this observer is listening for

Prior to Zen Cart 1.5.3, the update method is passed only three parameters. These are:

*   &$callingClass - This is a reference to the class in which the event occurred, allowing you access to that class's variables
*   $notifier - The name of the notifier that triggered the update (It is quite possible to observe more than one notifier)
*   $paramsArray - Made available in Zen Cart 1.5.x and represents data that is passed to the notifier for read access only.

NB! The observer/notifier system is written for an OOP-based application, as the observer expects to attach to a class that has notifiers within its methods. However a lot of the code within Zen Cart is still procedural in nature and not contained within a class.

To further help with managing the information made available by the notifier system and specifically to manipulate data within methods of classes where variables are local, as of Zen Cart 1.5.3 up to 8 additional variables have been made available in a standard install.  These additional variables are parameters made available to the notifier.

Another change that was introduced into Zen Cart 1.5.3 and above was the ability to execute a class method that is identified using the notifier name as part of the method name.  This is done by camelcasing the notifier which is to split the notifier name into the words between each underscore, capitalizing the first letter of each of those words and then combining them into a single word with the prefix of update.  So the method for the notifier `NOTIFIER_CART_ADD_CART_END` could be `updateNotifierCartAddCartEnd` which is to take `NOTIFIER_CART_ADD_CART_END` split it up into 5 words, capitalize each and combining to give `NotifierCartAddCartEnd` and then to add the prefix `update` to give `updateNotifierCartAddCartEnd`.  In Zen Cart 1.5.3 and above, if both the camelcased method and the `update` method exist in an observer class, then the camelcased method will be called.  If the notifier code is applied to a system that does not recognize the camelcased method of execution, then the `update` method will be called.

Therefore as of Zen Cart 1.5.3, the update method (or its camelcased observer method) is passed the following:

*   &$callingClass - This is a reference to the class in which the event occurred, allowing you access to that class's variables
*   $notifier - The name of the notifier that triggered the update (It is quite possible to observe more than one notifier)
*   $paramsArray - Made available in Zen Cart 1.5.x and represents data that is passed to the notifier for read access only.
*   &$param2 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param3 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param4 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param5 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param6 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param7 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param8 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.
*   &$param9 - Made available beginning in Zen Cart 1.5.3 and allows access to edit the variable that is passed.

Note that as of Zen Cart 1.5.3, if a parameter is not identified in the notifier that the value of the parameter will be null.  Prior to Zen Cart 1.5.3, no value will be passed and if the observer class method does not have a default value, then an error will be thrown and execution stopped.

To work around the procedural versus class monitoring of the notifier class, we added the 'stub' notifier class. So if you want to create an observer for a notifier that lies within procedural code (like in page headers) you should add the notifier into your myObserver class like this:

```
class myObserver extends base {
   function __construct() {
     global $zco_notifier;
     $zco_notifier->attach($this, array('NOTIFY_HEADER_END_CHECKOUT_CONFIRMATION'));
   }
```

As of Zen Cart 1.5.3, because $zco_notifier represents a class that extends the base class, the procedural code notifiers can be accessed the same way as those in other classes so the above can be written as:

```
class myObserver extends base {
   function __construct() {
     $this->attach($this, array('NOTIFY_HEADER_END_CHECKOUT_CONFIRMATION'));
   }
```

### Including observers into your code

Please note that the `includes/classes/observers` directory is not an autoload directory, so you will need to arrange for `application_top.php` to autoload your observer class as was described above (add a new `config.xxxxx.php` file in the `auto_loaders` folder, etc). Let's assume you are using the freeProduct class (see the example below), and you have saved this in `includes/classes/observers/class.freeProduct.php`.  The prefix to the file (e.g. `config.`) should align with what is assigned to `$loaderPrefix` within `includes/application_top.php` which is by default set to `config`.

You now need to arrange for this class to be loaded and instantiated. To do this you need to use the `application_top.php` autoload system.

In `includes/auto_loaders` create a file called `config.freeProduct.php` containing

```
$autoLoadConfig[10][] = array('autoType'=>'class',
                              'loadFile'=>'observers/class.freeProduct.php');
$autoLoadConfig[90][] = array('autoType'=>'classInstantiate',
                              'className'=>'freeProduct',
                              'objectName'=>'freeProduct');
```

Note: 10 has been chosen to cause the observer class to be loaded before the session is started. Note: 90 has been chosen as the offset since the observer (see the freeProduct example below) needs to be identified before the notifier (attached to `$_SESSION['cart']` instantiated at offset 80) is executed (e.g. in `includes/init_cart_handler.php`). If the observer was to execute within the construction/instatiation of the class, then it would need to load in a sequence before the class was instantiated.

To tie this all together, let's look at a real world example.

### A Real World Example

One of the most-often requested features is the ability for the store to automatically add a free gift to the Shopping Cart if the customer spends more than a certain amount.

The code has to be intelligent: it has to not only add the free gift when the shopper spends over a certain amount, but also remove the gift if the user changes the contents of the shopping cart, such that the total falls below the threshold.

Traditionally, although the code for this is not particularly difficult, it would have meant 'hacking' the core shoppingCart class in a number of places. With the ONS, this can be achieved with one very small custom class and absolutely no hacking whatsoever.

Here's the code.

```
<?php
/**
 * Observer class used to add a free product to the cart if the user spends more than $x
 *
 */
class freeProduct extends base {
  /**
   * The threshold amount the customer needs to spend.
   * 
   * Note this is defined in the shops base currency, and so works with multi currency shops
   *
   * @var decimal
   */
  var $freeAmount = 50;
  /**
   * The id of the free product.
   * 
   * Note. This must be a true free product. e.g. price = 0 Also make sure that if you don't want the customer
   * to be charged shipping on this, that you have it set correctly.
   *
   * @var integer
   */
  var $freeProductID = 57;
  /**
   * constructor method
   * 
   * Attaches our class to the $_SESSION['cart'] class and watches for 2 notifier events.
   */
  function __construct() {
    $_SESSION['cart']->attach($this, array('NOTIFIER_CART_ADD_CART_END', 'NOTIFIER_CART_REMOVE_END', 'NOTIFIER_CART_RESET_END'));
  }
  /**
   * Update Method
   * 
   * Called by observed class when any of our notifiable events occur
   *
   * @param object $class
   * @param string $eventID
   */
  function update(&$class, $eventID, $paramsArray = array()) {

  // If a product has been removed from the cart and the free product has been identified as being in the cart.
  if ($eventID == 'NOTIFIER_CART_REMOVE_END' && !empty($_SESSION['freeProductInCart']))
  {
    // If the free product is not in the cart, then it was the item that was removed, set a flag to indicate it was removed.
    if (!$_SESSION['cart']->in_cart($this->freeProductID))
    {
      $_SESSION['userRemovedFreeProduct'] = true;
    }
  }

  // If any notifier observed by this class has been triggered and there is no indication that the user removed the free product.
  //  Effectively this allows the customer to remove the product and have it not get auto-added back, but also to support auto-adding it.
  if (empty($_SESSION['userRemovedFreeProduct'])) 
  {
    // If the cart total meets the threshold for the free item and the free product is not in the cart:
    //   add it and set an indicator that it is present.
    if ($_SESSION['cart']->show_total() >= $this->freeAmount && !$_SESSION['cart']->in_cart($this->freeProductID) )   
    {
      $_SESSION['cart']->add_cart($this->freeProductID);
      $_SESSION['freeProductInCart'] = true;  
    }
  }

  // If any notifier observed by this class has been triggered and the threshold for the free item is not yet reached AND the free item is in the cart, then remove it.
  if ($_SESSION['cart']->show_total() < $this->freeAmount && $_SESSION['cart']->in_cart($this->freeProductID) ) 
  {
    $_SESSION['cart']->remove($this->freeProductID);
    unset($_SESSION['freeProductInCart']); // Free product is no longer in the cart.
    unset($_SESSION['userRemovedFreeProduct']); // User did not specifically request the product be removed, but was removed by circumstances thus allowing it to reappear when threshold met again.
  }

  // If any notifier observed by this class has been triggered and the free item is in the cart, be sure that quantity is set to a specific number.
  if ($_SESSION['cart']->in_cart($this->freeProductID)) 
  {
    $_SESSION['cart']->contents[$this->freeProductID]['qty'] = 1;
  }

  }  
}
?>
```

A couple notes:

First, I have set the options for the system in the class itself. This is obviously a bad idea, and it would be much better to have an admin module to set these options.

Second, notice that we are actually watching for three events in the one class.

```
$_SESSION['cart']->attach($this, array('NOTIFIER_CART_ADD_CART_END', 'NOTIFIER_CART_REMOVE_END', NOTIFIER_CART_RESET_END'));
```

so we are watching for the `NOTIFIER_CART_ADD_CART_END`, `NOTIFIER_CART_REMOVE_END`, and `NOTIFIER_CART_RESET_END` of the shopping_cart class.

The update class is extremely simple but in its simplicity manages to do all the work we require of it. It first tests to see if the total in the cart is over the threshold and, if it hasn't already, adds the free product to the cart. It also "remembers" if the customer has specifically removed the free product and if so, prevents auto-matically adding it back. Further when the content in the cart toggles around the threshold the product is removed and added back (although will do so even if the user had previously purposefully removed the product).

It then tests to see if the cart total has dropped below the threshold and, if the free product is in the cart, removes it and prepares to allow auto-adding it back when the cart total exceeds the threshold.

Now that was cool, how about something a little more difficult.

### Another Real World Example

Again we return to the Shopping Cart and promotions. Another oft-requested feature is the BOGOF promotion, or Buy One Get One Free. This is a little more difficult to achieve than our previous example, as there is some manipulation needed of the cart totals. However, as you will see it is still pretty much a breeze.

```
<?php
/**
 * Observer class used apply a Buy One Get One Free(bogof) algorithm to the cart
 *
 */
class myBogof extends base {
  /**
   * an array of ids of products that can be BOGOF.
   *
   * @var array
   */
  var $bogofsArray = array(10,4); //Under Siege2-Dark Territory & The replacement Killers
  /**
   * Integer number of bogofs allowed per product
   * 
   * For example if I add 4 items of product 10, that would suggest that I pay for 2 and get the other 2 free.
   * however you may want to restrict the customer to only getting 1 free regardless of the actual quantity
   *
   * @var integer
   */
  var $bogofsAllowed = 1;
  /**
   * constructor method
   * 
   * Watches for 1 notifier event, triggered from the shopping cart class.
   */
  function __construct() {
    $this->attach($this, array('NOTIFIER_CART_SHOW_TOTAL_END'));
  }
  /**
   * Update Method
   * 
   * Called by observed class when any of our notifiable events occur
   * 
   * This is a bit of a hack, but it works. 
   * First we loop through each product in the bogof Array and see if that product is in the cart.
   * Then we calculate the number of free items. As it is buy one get one free, the number of free items
   * is equal to the total quantity of an item/2.
   * Then we have to hack a bit (would be nice if there was a single cart method to return a product's in-cart price)
   * We loop thru the cart until we find the bogof item, get its final price, calculate the saving
   * and adjust the cart total accordingly.
   *
   * @param object $class
   * @param string $eventID
   */
  function update(&$class, $eventID) {
    $cost_saving = 0;
    $products = $_SESSION['cart']->get_products();
    foreach ($this->bogofsArray as $bogofItem) {
      if ($_SESSION['cart']->in_cart($bogofItem)) {
        if (isset($_SESSION['cart']->contents[$bogofItem]['qty']) && $_SESSION['cart']->contents[$bogofItem]['qty'] > 1) {
          $numBogofs = floor($_SESSION['cart']->contents[$bogofItem]['qty'] / 2);
          if ($numBogofs > $this->bogofsAllowed) $numBogofs = $this->bogofsAllowed;
          if ($numBogofs > 0) {
            for ($i=0, $n=sizeof($products); $i<$n; $i++) {
              if ($products[$i]['id'] == $bogofItem) {
                $final_price = $products[$i]['final_price'];
                break;
              }
            }
            $cost_saving .= $final_price * $numBogofs;
          }
        }
      }
    }
    $_SESSION['cart']->total -= $cost_saving;
  }
}
?>
```

NB: There are still some weaknesses here...

First although the adjust total is correctly shown on the shopping cart page and sidebox, the line total is not adjusted.

Secondly this will probably produce a confusing output at checkout.

Third: Have not tested for tax compliance yet ( @TODO )

Some development strategies include:
* in the `__construct` method, instead of listing a lengthy array grouping, the notifiers can be added to an array and that array be used in the attach method:
```
class myObserver extends base {
   function __construct() {
     $attachArray = array();
     $attachArray[] = 'NOTIFIER_CART_ADD_CART_END';
     $attachArray[] = 'NOTIFIER_CART_REMOVE_END';
     $attachArray[] = 'NOTIFIER_CART_RESET_END';
     
     $this->attach($this, $attachArray);
   }
   function update(&$callingClass, $notifier, $paramsArray) {
     ... do some stuff
   }
 }
 ```

OR as a single array statement.

```
class myObserver extends base {
   function __construct() {
     $attachArray = array(
                    'NOTIFIER_CART_ADD_CART_END',
                    'NOTIFIER_CART_REMOVE_END',
                    'NOTIFIER_CART_RESET_END',
                    )
     
     $this->attach($this, $attachArray);
   }
   function update(&$callingClass, $notifier, $paramsArray) {
     ... do some stuff
   }
 }
 ```
Either method allows adding/removing notifiers from the list in an understandable visually recognizable way that can simplify review/edit.


### Mods which support Notifier Use Development 
A number of mods are provided which can be helpful during development when using notifiers: 

* [Zen Cart Notifier Report](https://www.zen-cart.com/downloads.php?do=file&id=2260)
* [Zen Cart Notifier Trace](https://www.zen-cart.com/downloads.php?do=file&id=1114)

