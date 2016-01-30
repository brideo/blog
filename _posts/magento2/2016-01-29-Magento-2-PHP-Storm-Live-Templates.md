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
