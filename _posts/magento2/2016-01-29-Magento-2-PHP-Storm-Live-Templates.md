---
layout: post
title: PHP Storm Live Templates for Magento 2
categories: magento2
---

As part one in my using live templates in PHP Storm series, I have created some live templates to make basic taskes easier.

##Using Code Templates

In PHP Storm you have access to live templates, you should think of a code template as a shortcut for creating dynamic code snippets.

If you would like to create a new live template, you can write an example template; next you will want to select the code, double click shift and search for "Save as Live Template" which will take you to an interface for creating live templates.

You can use variables in your template, these will be for you to define when you make a snippet, if you use the same variable name twice it use your specified text in both instances. Variables are formatted like this: `$FOO$`.

Some code snippets I use which come out of the box with PHP Storm are things like `pubf` which will create:

	public function foo() {
		return new Bar();
	}

You also have access to `prof` and `prif` for `protected` and `private` functions.

For more information on live templates see Jefferey Way's screencast on [Live Templates.](https://laracasts.com/series/how-to-be-awesome-in-phpstorm/episodes/7)

##PHP Storm Live Templates for Magento 2

You can download these templates on my [Github](https://github.com/brideo/phpstorm-live-templates-magento-2)

To make my life easier during the transition to Magento 2 I have created a bunch of live templates.

These are very basic and I will be expanding on these, also it would be nice if the community include their live templates in this repo.

##Magento 2 Controller

Shortcode: `mcontroller`
{% highlight ruby %}
    <?php

namespace $NAMESPACE$\Controller\$ACTION$;

use Magento\Framework\App\Action\Action;
use Magento\Framework\App\Action\Context;
use Magento\Framework\View\Result\PageFactory;

class $CLASS$ extends Action
{

    /**
     * @var PageFactory
     */
    protected $resultPageFactory;

    /**
     * @var Context
     */
    protected $context;

    /**
     * Finder constructor.
     *
     * @param Context     $context
     * @param PageFactory $resultPageFactory
     */
    public function __construct(
        Context $context,
        PageFactory $resultPageFactory
    ) {
        parent::__construct($context);
        $this->resultPageFactory = $resultPageFactory;
        $this->context = $context;
    }

    /**
     * Load the page defined in view/frontend/layout/samplenewpage_index_index.xml
     *
     * @return \Magento\Framework\View\Result\\Magento\Framework\View\Result\Page
     */
    public function execute()
    {
        return $this->resultPageFactory->create();
    }
}
{% endhighlight %}

##Magento 2 Block

Shortcode: `mblock`

{% highlight ruby %}
    <?php

namespace $NAMESPACE$\Block;

use Magento\Framework\View\Element\Template;
use Magento\Framework\View\Element\Template\Context;

class $CLASS$ extends Template
{

    /**
     * Constructor
     *
     * @param Context $context
     * @param array   $data
     */
    public function __construct(
        Context $context,
        array $data = []$DEPENDENCY$
    ) {
        parent::__construct($context, $data);
    }
}
{% endhighlight %}

##Magento 2 Di.xml

Shortcode: `mdi`

{% highlight xml %}
    <?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="$NAMESPACE$\Api\$CLASSNAME$Interface" type="$NAMESPACE$\Service\$CLASSNAME$" />
</config>
{% endhighlight %}

##Magento 2 Basic layout.xml

Shortcode: `mlayout`


{% highlight xml %}
    <?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="content">
            <block class="$NAMESPACE$\$MODULE$\Block\$BLOCK$" name="$NAME$" template="$NAMESPACE$_$MODULE$::$TEMPLATE$.phtml" />
        </referenceContainer>
    </body>
</page>
{% endhighlight %}

##Magento 2 `module.xml`

Shortcode: `mmodule`

{% highlight xml %}
    <?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="$MODULE$" setup_version="1.0.0" />
</config>
{% endhighlight %}

##Magento 2 `routes.xml`

Shortcode `mroutes`

{% highlight xml %}
    <?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="$AREA$">
        <route id="$FRONTNAME$" frontName="$FRONTNAME$">
            <module name="$MODULE$" />
        </route>
    </router>
</config>
{% endhighlight %}

##Magento 2 `system.xml`

Shortcode `msystem`

{% highlight xml %}
    <?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="$MOODULE_NAME$" translate="label" type="text" sortOrder="300" showInDefault="1" showInWebsite="1" showInStore="1">
            <label>$LABEL$</label>
            <tab>$TAB$</tab>
            <resource>$RESOURCE$</resource>
            <group id="$MOODULE_NAME$" translate="label" type="text" sortOrder="300" showInDefault="1" showInWebsite="0" showInStore="0">
                <label>$LABEL$</label>
                <field id="$FIELD_ID$" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>$FIELD_LABEL$</label>
                    <source_model>$SOURCE_MODEL$</source_model>
                </field>
            </group>
        </section>
    </system>
</config>
{% endhighlight %}

##Magento 2 `acl.xml`
 
 Shortcode `macl`

{% highlight xml %}
    <?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="Magento_Backend::content">
                    <resource id="$NAMESPACE$::$ACTION$" title="$TITLE$" sortOrder="10" />
                </resource>
            </resource>
        </resources>
    </acl>
</config>
{% endhighlight %}

##Magento 2 `menu.xml`
 
 Shortcode `mmenu`
 
 {% highlight xml %}
    <?xml version="1.0"?>
 <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
     <menu>
         <add id="$NAMESPACE$::$ACTION$" action="$URL$" title="$TITLE$" module="$NAMESPACE$" sortOrder="100" parent="Magento_Backend::content" resource="$NAMESPACE$::$ACTION$" />
     </menu>
 </config>
 {% endhighlight %}

##Magento 2 Block Grid Container

Shortcode `mgrid`

{% highlight ruby %}
    <?php

namespace $COMPANY$\$MODULE$\Block\Adminhtml;

use Magento\Backend\Block\Widget\Grid\Container;

class $CLASS$ extends Container
{
    protected function _construct()
    {
        $this->_controller = 'adminhtml_$CONTROLLER$';
        $this->_blockGroup = '$COMPANY$_$MODULE$';
        $this->_headerText = __('$TITLE$');
        parent::_construct();
    }

}
 {% endhighlight %}

##Magento 2 Unit Test

{% highlight ruby %}
<?php

namespace $NAMESPACE$;

use PHPUnit_Framework_TestCase;

class $CLASS$Test extends PHPUnit_Framework_TestCase
{


}
{% endhighlight %}
 
##Magento 2 Backend Controller
 
 Shortcode `mbcontroller`
 
 {% highlight ruby %}
    <?php
 namespace $COMPANY$\$MODULE$\Controller\Adminhtml\$DIR$;
 
 use Magento\Backend\App\Action as BackendAction;
 use Magento\Backend\App\Action\Context;
 use Magento\Framework\View\Result\PageFactory;
 
 class $ACTION_NAME$ extends BackendAction
 {
 
     const ADMIN_RESOURCE = '$COMPANY$_$MODULE$::$ACTION_LOWER$';
 
     const PAGE_TITLE = '$PAGE_TITLE$';
 
     /**
      * @var PageFactory
      */
     protected $resultPageFactory;
 
     /**
      * @param Context $context
      * @param PageFactory $resultPageFactory
      */
     public function __construct(
         Context $context,
         PageFactory $resultPageFactory
     ) {
         parent::__construct($context);
         $this->resultPageFactory = $resultPageFactory;
     }
 
     /**
      * Index action
      *
      * @return \Magento\Framework\View\Result\Page
      */
     public function execute()
     {
         /** @var \Magento\Framework\View\Result\Page $resultPage */
         $resultPage = $this->resultPageFactory->create();
         $resultPage->setActiveMenu(static::ADMIN_RESOURCE);
         $resultPage->addBreadcrumb(__(static::PAGE_TITLE), __(static::PAGE_TITLE));
         $resultPage->getConfig()->getTitle()->prepend(__(static::PAGE_TITLE));
 
         return $resultPage;
     }
 
     /**
      * Is the user allowed to view the blog post grid.
      *
      * @return bool
      */
     protected function _isAllowed()
     {
         return $this->_authorization->isAllowed(static::ADMIN_RESOURCE);
     }
 
 
 }
{% endhighlight %}

##Magento 2 jQuery Plugin Template
 
Shortcode `mjquery`
 
{% highlight javascript %}
    ;require(['jquery'], function ($) {

    /**
     * Declare a namespace if it doesn't already
     * exist.
     */
    if (!$.$COMPANY$) {
        $.$COMPANY$ = {};
    }

    /**
     * Colour switching plugin declaration.
     *
     * @param element
     * @param  options
     * @constructor
     */
    $.$COMPANY$.$MODULE$ = function (element, options) {

        /**
         * This variable is used because it's
         * easier than binding `this`
         *
         * @type {$.$COMPANY$.$MODULE$}
         */
        var base = this;

        /**
         * Accessor to the element we are manipulating.
         *
         * @type {*|jQuery|HTMLElement}
         */
        base.$element = $(element);
        base.element = element;

        /**
         * Add a reverse reference to the plugin.
         */
        base.$element.data("$COMPANY$.$MODULE$", base);

        /**
         * Plugin constructor
         */
        base.init = function () {
            base.options = $.extend({}, $.$COMPANY$.$MODULE$.defaultOptions, options);

        };

        /**
         * Initialize the plugin.
         */
        base.init();
    };

    /**
     * Default options for the plugin.
     *
     * @type
     */
    $.$COMPANY$.$MODULE$.defaultOptions = {};

    /**
     * Setup the plugin, this will override the default
     * options with the object passed in instantiation.
     *
     * @constructor
     *
     * @param options
     */
    $.fn.$COMPANY$_$MODULE$ = function
        (options) {
        return this.each(function () {
            (new $.$COMPANY$.$MODULE$(this, options));
        });
    };

});
{% endhighlight %}
