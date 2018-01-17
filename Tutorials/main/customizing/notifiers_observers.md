# Notifiers and observers, how they work and how to use them.
## Notifiers:
Notifiers are points in code that during execution provide a point where an observer can take an action.
For those that have previous code experience in other languages, a notifier is initially like a gosub routine that may have
been seen in older programming languages like BASIC.  In more recent languages such as C, C++, etc... a notifier works like
calling some "external" function.  The thing is that there does not have to be just one function that listens (or observes) that
notifier.

Observers are the code that is executed when the identified notifier has been reached. There can be more than one observer for
any identified notifier and there are several ways available to sequence each observer so that they may load/execute in a 
specific sequence.

The notifier/observer system has been a part of Zen Cart from its early 1.3.x days.  The usability and flexibility of that system
has only really been expanded since Zen Cart 1.5.3.

## What are the parts of a notifier?

Here is a notifier that is fully supported in Zen Cart 1.5.3 and above with it used in what is called the global space:
`$zco_notifier->notify('NOTIFIER_NAME', array('val_i'=>$i, 'val_j'=>$j), $i, $j);`

This includes an object `$zco_notifier` that is the variable name given to the notifier class which is also a member of the base class.
A method of the notifier class `notify`.
The notifier: 'NOTIFIER_NAME'
A non-editable "variable" which could be made of an array to support passing multiple values or a single variable: 
`array('val_i'=>$i, 'val_j'=>$j)` or just `$k`.
And two instances of a variable that when properly received can be edited within the observer. <- Added in Zen Cart 1.5.3.

If the same notifier is used in a pre-Zen Cart 1.5.3 environment, the editable values shown will not be made directly available
to the observer and must be pulled from the global space in order to edit them.

The notifier portion has always supported both first the "name" of the notifier and then a first value that could be passed
to the observer as an array:
`$zco_notifier->notify('NOTIFIER_NAME', array('val_i'=>$i, 'val_j'=>$j));`

or just as a value:
`$zco_notifier->notify('NOTIFIER_NAME_OTHER', $k);`

As of Zen Cart 1.5.3, it has been possible to also pass additional information that can then be modified from within the observer.

`$zco_notifier->notify('NOTIFIER_NAME', array('val_i'=>$i, 'val_j'=>$j), $i /*editable*/, $j /*editable*/);`

The third and fourth parameters of the above notifier ($i, $j) are editable if the observer is properly written to allow editing
the value(s).  This means that as far as variable scope that a value that is perhaps not in the global space can be edited 
within a function and the result of that edit will be carried on in program execution.

Observers:

The observer is a function within a class that will execute the applicable code when the notifier has been triggered or 
encountered within code execution.


