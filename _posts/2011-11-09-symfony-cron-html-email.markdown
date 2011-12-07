---
layout: post
title: Symfony, Cron and HTML Emails (getPartial from cron)
---
I googled and I googled and I googled some more. I couldn’t find a definite answer to this. From the post title you can probably tell that I am wanting to write a cron job that performs some sort of logic and sends emails. Yes. More specifically, I am writing a cron job to check a schedule of all users in a system, and then send them a notification if they have unconfirmed schedule entries. I want the email being sent to an html email generated from a partial.

But wait!!

getPartial is only available in from sfAction! What do I do? Well after googling this for a bit I could not find any definite answer. So I looked at the source code of sfAction (because open source is awesome) and found that sfActions getPartial method simply loads the partial helper and calls get_partial(), which is typically used from a template to get the contents of a partial as a string.

So my cron job is set up as follows:

{% highlight php %}
require_once(dirname(__FILE__).'/../config/ProjectConfiguration.class.php');
 
$configuration = ProjectConfiguration::getApplicationConfiguration('frontend', 'dev', true);
sfContext::createInstance($configuration); // <-- note that this looks the same as the regular Symfony front controller except you remove the call to dispatch that is usually here
{% endhighlight %}

This is saved in the Symfony root in a batch folder. You can call the script whatever you want.

At this point in the script execution you have full access to all the wonderful included in Symfony.

So to get to the point of this post here is how I managed to use partials in a cron job.

{% highlight php %}
require_once(dirname(__FILE__).'/../config/ProjectConfiguration.class.php');
 
$configuration = ProjectConfiguration::getApplicationConfiguration('frontend', 'dev', true);
sfContext::createInstance($configuration);
 
// this loads in the Partial helpers
sfContext::getInstance()->getConfiguration()->loadHelpers('Partial');
 
// retrieve users from the database
$users = Doctrine_Core::getTable('Users')->findByCondition($someCondition);
 
foreach ($users as $user)
{
  $mailer = sfContext::getInstance()->getMailer();
  $message = $mailer->compose(array('from@domain.com'=>'From Name'), array('to@domain.com'=>'To Name'), 'Email Subject');
  $emailBody = get_partial('module/notifications',array('user'=>$user)); // use get_partial as you normally would
  $message->setBody($emailBody,'text/html');
  $mailer->send($message);
}
{% endhighlight %}

Worked pretty well, I have some query optimizations I’d probably want to do in my particular implementation, but that is beyond the scope of this post.
