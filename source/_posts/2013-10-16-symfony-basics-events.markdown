---
layout: post
title: "Symfony Basics: Events"
date: 2013-10-16 00:31
comments: true
categories: Symfony
---
In this new post I'll talk about a Symfony component which is used to help you users easily hook into the flow of your code. This is done by the [EventDispatcher](https://github.com/symfony/EventDispatcher).

### Why do I need events?

Event dispatching allows you to develop a library which can easily be adapted by it's users for their own needs. For instance, they can log certain actions when they happen, or can modify an object before you use it. I will show how this is done with the EventDispatcher component in the next section.

### Using the EventDispatcher component

So, if the reasoning behind why you should use event dispatching convinced you, let's see how this is used. All the code used here is available in my [event dispatcher example github repository](https://github.com/Lumbendil/EventDispatcherExample).

What we will do is a fake mailer, which will be given a list of users and send each of them a text. Instead of sending emails, we'll print the email which would be sent through the screen.

So, the first thing we need to do is create the mailer class.

```php
class WelcomeMailer
{
	/**
	 * @param string[] $emails An array of emails.
	 */
	public function sendEmails(array $emails)
	{
		foreach ($emails as $email) {
			$this->sendEmail($email);
		}
	}
	
	/**
	 * @param string $email The email of the user.
	 * @return bool True if the email was sent successfully,
	 * 	false otherwise.
	 */
	private function sendEmail($email)
	{
		$message = "Welcome $email";
		echo "Sending message to $email: $message", PHP_EOL;
		return true;
	}
}

$mailer = new WelcomeMailer();
$emails = array(
	'bob@example.com',
	'alice@example.com',
);
$mailer->sendEmails($emails);
```

It's a really nice and simple class, and, if it actually sent emails and sent a more elaborate email, it could be useful for some people. But someone could, for instance, need to send a special email to some users, need to know how many successful and unsuccessful emails happened and log that, etc. Even though this can be accomplished in several ways, we'll show how to do it with events.

First of, we need the mailer to depend on the event dispatcher. If the welcome mailer was a service in your application, this could be done using [dependency injection](lumbendil.github.io/blog/2013/10/02/symfony-bases-services/).

```php
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use Symfony\Component\EventDispatcher\EventDispatcher;

class WelcomeMailer
{
	private $eventDispatcher;
	
	public function __construct(EventDispatcherInterface $eventDispatcher)
	{
		$this->eventDispatcher = $eventDispatcher;
	}
	
	// ...
}

$dispatcher = new EventDispatcher();
$mailer = new WelcomeMailer($dispatcher);

// ...
```

So, once we have the event dispatcher in place, it's time for us to start dispatching events. To ease this up, we'll use the class GenericEvent, to avoid having to write to many classes. Even though, it'd be better to have a class for each event.

```php
use Symfony\Component\EventDispatcher\GenericEvent;

// ...

	public function sendEmails(array $emails)
	{
		$this->eventDispatcher->dispatch('welcome_mailer.mailer_start');

		foreach ($emails as $email) {
			$event = new GenericEvent();
			$event['email'] = $email;

			if ($this->sendEmail($email)) {
				$this->eventDispatcher->dispatch('welcome_mailer.send_successful', $event);
			} else {
				$this->eventDispatcher->dispatch('welcome_mailer.send_failed', $event);
			}
		}

		$this->eventDispatcher->dispatch('welcome_mailer.mailer_end');
	}
	
	/**
	 * @param string $email The email of the user.
	 * @return bool True if the email was sent successfully,
	 * 	false otherwise.
	 */
	private function sendEmail($email)
	{
		$message = "Welcome $email";

		$event = new GenericEvent();
		$event['email'] = $email;
		$event['message'] = $message;
		
		$this->dispatcher->eventDispatcher('welcome_mailer.mail_prepared', $event);
		$message = $event['message'];

		echo "Sending message to $email: $message", PHP_EOL;
		return true;
	}
```

So now, we have added five events. The `mailer_start`, `mailer_end`, `send_successful`, `send_failed` and `mail_prepared` events. We have an object which represents the event, and is used to communicate things between the event dispatcher and the library. So, let's see how could we use this events. Let's see how we can use this events to handle the couple of examples we mentioned earlier.

A simple example would be to customize the message we are sending to `alice@example.com`, so it says `Hi Alice.`.

```php
class AliceEmailListener
{
	public function changeAliceMessage(GenericEvent $event)
	{
		if ($event['email'] == 'alice@example.com') {
			$event['message'] = 'Hi Alice.';
		}
	}
}

$dispatcher->addListener('welcome_mailer.mail_prepared', array(new AliceEmailListener(), 'changeAliceMessage'));
```

So, we have implemented a listener, which listens to the mail prepared event, and when it is triggered, it'll change the message if it's Alice's email. The listener here has been given as an array of an object and a function name, but it will work as long as it's a valid callable.

For the next part, counting successful and failed emails, we'll use a subscriber. A subscriber is a class which implements a given interface, the `EventSubscriberInterface`, which has a method to get all the events the subscriber wants to be notified at.

```php
use Symfony\Component\EventDispatcher\GenericEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class EmailCountSubscriber implements EventSubscriberInterface
{
	private $successes;
	private $failures;

	public static function getSubscribedEvents()
	{
		return array(
			'welcome_mailer.mailer_start' => 'resetCounters',
			'welcome_mailer.mailer_end' => 'printCounters',
			'welcome_mailer.send_successful' => 'countSuccess',
			'welcome_mailer.send_failed' => 'countFailure',
		);
	}
	
	public function resetCounters()
	{
		$this->successes = $this->failures = 0;
	}
	
	public function countSuccess()
	{
		$this->successes++;
	}
	
	public function countFailuer()
	{
		$this->failures++;
	}
	
	public function printCounters()
	{
		echo "Mailing ended. Successes: {$this->successes}. Failures: {$this->failures}.", PHP_EOL;
	}
}

$dispatcher->addSubscriber(new EmailCountSubscriber());
```

### Conclusion

There are some things I haven't covered in this post, such as priorities and stopping event propagation. In order to look more onto it, you should go to [the component documentation](http://symfony.com/doc/current/components/event_dispatcher/index.html).