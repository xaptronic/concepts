---
layout: post
title: Keeping branding consistent across multiple symfony 1.4 installations (Using layouts from plugins)
---
How do you keep branding consistent across many symfony 1.4 installations? This was recently asked on the symfony mailing list (symfony-users@googlegroups.com). The poster suggested having a plugin that is installed into each symfony app that contained a layout to use for all views.

I looked into this and here is what I found.

When symfony goes to render a template, it uses

{% highlight php %}
sfConfig::get('sf_app_template_dir')
{% endhighlight %}

to determine where to look for the decorator directory. The sfApplicationConfiguration is responsible for serving this to any callers and therefore it by default points to the current application’s templates directory.

Fine and dandy, however how do we get it to point to a template held in a plugin? Well the funny thing is that its pretty easy, however it is rather inflexible at the same time.

I was able to add a quick line to my project configuration file to point this at a plugin directory.

For example:

{% highlight php %}
require_once dirname(__FILE__).'/../lib/vendor/symfony/lib/autoload/sfCoreAutoload.class.php';
sfCoreAutoload::register();
class ProjectConfiguration extends sfProjectConfiguration
{
  public function setup()
  {
    // this sets the template dir to my plugins templates
    sfConfig::set('sf_app_template_dir', sfConfig::get('sf_plugins_dir').'/xTestPlugin/templates');
    $this->setWebDir(sfConfig::get('sf_root_dir').'/content');
    $this->enablePlugins(array(
      'sfDoctrinePlugin',
      'sfWebBrowserPlugin',
      'sfFeed2Plugin',
      'xTestPlugin', // dont think this needs to be here for this simple case, but if this plug does other things it will likely be needed
    ));
  }
}
{% endhighlight %}

Ok, looks good. How is this inflexible? Well this method in sfApplicationConfiguration

{% highlight php %}
public function getDecoratorDirs()
{
  return array(sfConfig::get('sf_app_template_dir'));
}
{% endhighlight %}

Makes it difficult to set up an array of decorator dirs, because if you pass set sf_app_template_dir to an array then it gets made into an array of arrays which breaks the function that gets the decorator dir for a particular template. So it works if I only want to put my template directories in one place, but if I want to use both a plug in and the templates directory, I ended up overriding this method in the application configuration class.

{% highlight php %}
class frontendConfiguration extends sfApplicationConfiguration
{
  public function configure()
  {
  }
 
  public function getDecoratorDirs()
  {
    return sfConfig::get('app_template_dirs', parent::getDecoratorDirs());
  }
}
{% endhighlight %}

This way by default, it will use whatever symfony uses as the default, otherwise I can specify my own array (MUST BE AN ARRAY) of decorator dirs in my app.yml, and it symfony will search through all of them.

Hopefully this will help, and I just saw someone post an alternative solution on the mailing list. I imagine there is probably an even better way than this, however this is flexible and allows you to have templates in multiple places. Just watch for having templates named the same, as it will take the first one it finds.
