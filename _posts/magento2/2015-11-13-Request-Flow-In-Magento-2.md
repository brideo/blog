---
layout: post
title: Request Flow in Magento 2
categories: magento2
---

- Index.php
- Bootstrap
    - `Bootstrap::run()`
- App
    - `App:launch()`
- Routing
    - `FrontController::dispatch()`
    - `Router::match()`
- Controller Processing
    - `Controller::execute()`
- Rendering
    - `View::loadLayout()`
    - `View::renderLayout()`
- Flushing Output
    - `Response::sendResponse()`

If you are familiar with Magento 1.x, you should be able substantial similarities in the request flow in Magento 2 is. 

Now Magento 2 is built on composer the request flow is slightly different, however the fundamentals are almost the same regarding request flow.
  
##Initiation Phase in Magento 2

As always the entry point for Magento 2 is in the root level `index.php`. Another entry point accessible to us are `cron.php`.

The entry point for Magento now looks like this:

{% highlight ruby %}
    $bootstrap = \Magento\Framework\App\Bootstrap::create(BP, $_SERVER);
/** @var \Magento\Framework\App\Http $app */
$app = $bootstrap->createApplication('Magento\Framework\App\Http');
$bootstrap->run($app);
{% endhighlight %}

This is important to remember if you ever need to create your own entry point, for debugging or running quick queries.

There are two `index.php` files in Magento:

- `/var/www/magento2/index.php` - Development
- `/var/www/magento2/pub/index.php` - Production

##Routing Magento 2

A router is tasked with taking a pretty SEO friendly URL path and converting it to a Magento 2, setting any parameters available to the request and identifying the appropriate controller to handle the request.

A big difference with controller classes now in Magento is rather than having an `actionMethod()` for a single url, you now have an action class. Your action class should contain an `execute()` method which handles the page rendering and output.

URL's are the same as in Magento 1.x as in you `/frontName/actionPath/action/param1/value1/param2/value2`.

The method `Magento\Framework\App\Router\Base::matchAction()` is very insightful section into how this works:

{% highlight ruby %}
    /**
    * Going through modules to find appropriate controller
*/
$currentModuleName = null;
$actionPath = null;
$action = null;
$actionInstance = null;

$actionPath = $this->matchActionPath($request, $params['actionPath']);
$action = $request->getActionName() ?: ($params['actionName'] ?: $this->_defaultPath->getPart('action'));
$this->_checkShouldBeSecure($request, '/' . $moduleFrontName . '/' . $actionPath . '/' . $action);

foreach ($modules as $moduleName) {
    $currentModuleName = $moduleName;

    $actionClassName = $this->actionList->get($moduleName, $this->pathPrefix, $actionPath, $action);
    if (!$actionClassName || !is_subclass_of($actionClassName, $this->actionInterface)) {
        continue;
    }

    $actionInstance = $this->actionFactory->create($actionClassName);
    break;
}
{% endhighlight %}

##Front Controller

The front controller gathers routes and selects a controller using DI. Although Magento 2 still has rewrites, pretty url's, these are gathered elsewhere. The `FrontController` class only has one method and implements the `FrontControllerInterface` interface. 

##Routing Mechanism

Here are the 5 routers in Magento and the order in which they get fired in:

- Base Router
    - `Magento\Core\App\Router\Base`
- CMS Router
    - `Magento\Cms\Controller\Router`
- URL Rewrite Router
    - `Magento\UrlRewrite\Controller\Router`
- Design Editor
- `Magento\DesignEditor\Controller\Varien\Router\Standard`
- Default Router (404 Router)
    - `Magento\Framework\App\Router\DefaultRouter`

An example of routing a non-standard router can be found in `Magento\Cms\Controller\Router.php`:

{% highlight ruby %}
    $request->setModuleName('cms')->setControllerName('page')->setActionName('view')->setParam('page_id', $pageId);
{% endhighlight %}
    
Each request is sent through the series of routers, it uses the `Router::match()` method as a flag, if it's false we continue to the next router, if it's true we run `Controller::dispatch($action)`.

You can add your own custom router by adding it to `Magento\Framework\App\RouterList`.

Magento 2 checks the the assigned list of modules, foreach module it builds a potential action class and then checks whether the action class exists; if it exists it returns the class, else it will repeat. 

##Controller Architecture

As mentioned before, we now only have one action per controller **Action Class**, an action class should contain:

* `execute()` method
* Constructor where you set up you DI (Dependency Injection)

Action controllers extend `Magento\Framework\App\Action\Action` which in turn extends `Magento\Framework\App\Action\Abstract`, the abstract class implements the `\Magento\Framework\App\ActionInterface`.

The `ActionInterface` class only contains one method:

* `public function execute();`

The `Action\Abstract` class contains the request and response objects, you will set these up using `parent::__construct()` (DI), as before the methods are:

* `getRequest()`
* `getResponse()`

The `Action\Action` class contains the `dispatch()` method, this is the method that the router will call; this in turn calls the `execute()` method.

The Catalog and Sales modules inheritance is slightly different in that they extend a class which extends `Action\Action`.

##Admin and Frontend Controllers

As with Magento 1.x admin controllers are protected using ACL. Rather than having an `adminhtml` folder like in Magento 1.x we now have a `backend` folder.

Magento backend controllers extend the `Magento\Backend\App\Action` extends `\Magento\Backend\App\AbstractAction`, this then extends the `\Magento\Framework\App\Action\Action` class mentioned earlier.

The `\Magento\Backend\App\AbstractAction` rewrites the `dispatch()` method, the main thing to note is that the new `dispatch()` calls `_isAllowed()`. If you're coming from a Magento 1.x background you should be familiar with this method by now. The `_isAllowed()` method is very important, it is the method which tells Magento whether the current user is aloud on the current admin route.

{% highlight ruby %}
    /**
 * @return bool
 */
protected function _isAllowed()
{
    return $this->_authorization->isAllowed(self::ADMIN_RESOURCE);
}
{% endhighlight %}

You will need to implement the `_isAllowed()` method within your admin action class.

Changes the backend `AbstractAction` class implement are as follows:

- Extra DI
- Rewritten `dipatch()`
- `_isAllowed()` method.
- Rewritten `redirect()` method
- Rewritten `forward()` method
- Extra auxilary methods
    - `_getSession()`
    - `_addBreadcrumb()`
    - `_addJs()`
    - `_addContent()`
    - `_addLeft()`
    - `_getUrl()`
    
Yet again, these should be familiar if you're coming from Magento 1.x.

###Steps in debugging:

- Check if the `frontName` is correct
- Check the list of available module.s
- Check the path name foreach module

###Creating a controller

To create a controller you must create a `etc/{area}/routes.xml` file within your project, create an action class with the `execute()` method, then of course **WRITE YOUR TESTS**.

The two areas are:

- frontend
- adminhtml

Your `routes.xml` file should look like this:
{% highlight xml %}
    <?xml version="1.0"?>
<!--
/**
 * Copyright © 2015 Magento. All rights reserved.
 * See COPYING.txt for license details.
 */
-->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="{area_id}">
        <route id="{module_name}" frontName="{front_name}">
            <module name="{Namespace_Modulename}" />
        </route>
    </router>
</config>
{% endhighlight %}
The two `area_id`'s are:

- standard
- admin
    
Earlier we mentioned how the url is created from 3 sections (frontName, ActionPath, Action), here is how they relate to your controllers:

- Front Name
    - Usually your module name, defined in XML
- ActionPath
    - Sub-folder of the Controller folder of the module
- Action
    - The Controller Class

