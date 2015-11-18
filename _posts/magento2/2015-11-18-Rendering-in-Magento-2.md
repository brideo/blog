---
layout: post
title: Rendering in Magento 2
categories: magento2
---

Magento 1.x and Magento 2 rendering is almost identical conceptually. The application will gather all of the modules configuration XML, next it will generate the layout XML which in turn renders to blocks; after this blocks add the templates and render any child blocks within them, finally `toHtml()` is called and the it flushes the output.

##Rendering our Response.

`Magento\Framework\App\View` is the main class involved in rendering layout in Magento, this class implements the `ViewInterface`.

The methods which the view interface contain are:

- `public function loadLayoutUpdates()`
- `public function renderLayout($output = '')`
- `public function getDefaultLayoutHandle()`
- `public function loadLayout($handles = null, $generateBlocks = true, $generateXml = true, $addActionHandles = true)`
- `public function generateLayoutXml()`
- `public function addPageLayoutHandles(array $parameters = [], $defaultHandle = null)`
- `public function generateLayoutBlocks()`
- `public function getPage()`
- `public function getLayout()`
- `public function addActionLayoutHandles()`
- `public function setIsLayoutLoaded($value)`
- `public function isLayoutLoaded()`

Magento 1.x developers will remember that controllers had the `loadLayout()` and `renderLayout()` in the controllers, Magento 2 action controllers don't have this anymore and the logic has been moved to the View Object.

The `Magento\Framework\View\Layout` is the equivalent to `Mage_Core_Model_Layout` class in Magento 1.x, this class is for rendering and generating the layout XML files within Magento.

The `Magento\Framework\View\Layout` extends the `\Magento\Framework\Simplexml\Config` class and implements the `\Magento\Framework\View\LayoutInterface` interface.

I'm not going to list the methods in this interface as there is many, however you should take a look.

In Magento 1.x the controller would load the layout, `loadLayout()`, whereas in Magento 2 the controller creates the page object and sends them back to `App::launch()`. Only one method is used for rendering and loading the layout in Magento 2 `Page::renderResult()`.

Next, Magento 2 calls `Page::render()`, this method calls `$this->pageConfig->publicBuild();` which is a wrapper for `Magento\Framework\View\Page\Config::build()`:

{% highlight ruby %}
/**
 * Build page config from page configurations
 * @return void
 */
protected function build()
{
    if (!empty($this->builder)) {
        $this->builder->build();
    }
}
{% endhighlight %}

The `builder` object is injected using the DI method (dependency injection) however you can override it using `Config::setBuilder(BuilderInterface $builder)`.

Magento's default builder object, `Magento\Framework\View\Page\Builder` extends `View\Layout\Builder`, these classes contain methods which are used in Magento 1.x also:

- `generateLayoutBlocks()`
- `generateLayoutXml()`
- `loadLayoutUpdates()`
- `getPageLayout()`

`View\Layout\Builder` implements the `BuilderInterface`, this only contains a single method: `build()`, the `View\Layout\Builder::build()` method looks like this:

{% highlight ruby %}
    /**
 * Build layout structure
 *
 * @return \Magento\Framework\View\LayoutInterface
 */
public function build()
{
    if (!$this->isBuilt) {
        $this->isBuilt = true;
        $this->loadLayoutUpdates();
        $this->generateLayoutXml();
        $this->generateLayoutBlocks();
    }
    return $this->layout;
}
{% endhighlight %}

Magento 1.x developers should see how old logic is wrapped in a shiny new interface now, cool huh?

`Page::renderPage()` includes the template set against the object, then includes the file within the `ob_start()` tags.


##View Elements

###UiComponent

Magento UI Components allow for easily rendering and altering UI elements.

UiComponents refers to the `Magento\Ui` module, I recommend looking through the `Component` directory in that module.

Some UiComponents are:

- Form Field
- Search
- Text
- Paging
- Select

The interfaces and meta classes are in `Magento\Framework\View\Element`; the `UiComponentInterface` interface extends the `BlockInterface`.

The `UiComponentInterface` has the following methods:

- `public function getName()`
- `public function getComponentName()`
- `public function getConfiguration()`
- `public function render()`
- `public function getComponent($name)`
- `public function getChildComponents()`
- `public function getTemplate()`
- `public function getContext()`
- `public function renderChildComponent($name)`
- `public function setData($key, $value = null)`
- `public function getData($key = '', $index = null)`
- `public function prepare()`
- `public function prepareDataSource(array $dataSource)`
- `public function getDataSourceData()`

UiComponents are rendered using `toHtml()` much like blocks, check the `Magento\Framework\View\Layout::_renderUiComponent()` if you're interested.

###Containers

A container renders all of it's children automatically just like a `core/text_list` block. You can't create an instance of a container because they're an abstract concept, however you can create an instance of a block, this can be seen in `Magento\Framework\View\Layout::_renderContainer($name)`:

{% highlight ruby %}
    /**
 * Gets HTML of container element
 *
 * @param string $name
 * @return string
 */
protected function _renderContainer($name)
{
    $html = '';
    $children = $this->getChildNames($name);
    foreach ($children as $child) {
        $html .= $this->renderElement($child);
    }
    if ($html == '' || !$this->structure->getAttribute($name, Element::CONTAINER_OPT_HTML_TAG)) {
        return $html;
    }

    $htmlId = $this->structure->getAttribute($name, Element::CONTAINER_OPT_HTML_ID);
    if ($htmlId) {
        $htmlId = ' id="' . $htmlId . '"';
    }

    $htmlClass = $this->structure->getAttribute($name, Element::CONTAINER_OPT_HTML_CLASS);
    if ($htmlClass) {
        $htmlClass = ' class="' . $htmlClass . '"';
    }

    $htmlTag = $this->structure->getAttribute($name, Element::CONTAINER_OPT_HTML_TAG);

    $html = sprintf('<%1$s%2$s%3$s>%4$s</%1$s>', $htmlTag, $htmlId, $htmlClass, $html);

    return $html;
}
{% endhighlight %}

You can see no instance is created of the container, it's just rendered along with it's children; which could be another container. You have the option of adding classes and ID's your container. You define your containers in XML just like with everything in Magento. A great example of this is in `app/code/Magento/Theme/view/base/page_layout/empty.xml`:

{% highlight xml %}
    <?xml version="1.0"?>
<!--
/**
 * Copyright © 2015 Magento. All rights reserved.
 * See COPYING.txt for license details.
 */
-->
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <container name="root">
        <container name="after.body.start" as="after.body.start" before="-" label="Page Top"/>
        <container name="page.wrapper" as="page_wrapper" htmlTag="div" htmlClass="page-wrapper">
            <container name="global.notices" as="global_notices" before="-"/>
            <container name="main.content" htmlTag="main" htmlId="maincontent" htmlClass="page-main">
                <container name="columns.top" label="Before Main Columns"/>
                <container name="columns" htmlTag="div" htmlClass="columns">
                    <container name="main" label="Main Content Container" htmlTag="div" htmlClass="column main"/>
                </container>
            </container>
            <container name="page.bottom.container" as="page_bottom_container" label="Before Page Footer Container" after="main.content" htmlTag="div" htmlClass="page-bottom"/>
            <container name="before.body.end" as="before_body_end" after="-" label="Page Bottom"/>
        </container>
    </container>
</layout>
{% endhighlight %}

In Magento 1.x we had a `1column.phtml` template, in Magento 2 we have `1column.xml` which updates containers from `empty.xml` in turn creating a 1 column layout:

{% highlight xml %}
    <layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
        <update handle="empty"/>
        <referenceContainer name="page.wrapper">
            <container name="header.container" as="header_container" label="Page Header Container"  htmlTag="header" htmlClass="page-header" before="main.content"/>
            <container name="page.top" as="page_top" label="After Page Header" after="header.container"/>
            <container name="footer-container" as="footer" before="before.body.end" label="Page Footer Container" htmlTag="footer" htmlClass="page-footer" />
        </referenceContainer>
    </layout>
{% endhighlight %}

I find this pretty exciting as if you got to grips with layout XML in Magento 1.x you'll know how powerful it is.

##Templates

Just like in Magento 1.x, the default rendering system is `.phtml`. However, good news for front-ends in Magento 2 we can easily use any rendering system.

The Magento 2 fallback system is now done in `Block::getTemplateFile()`:

{% highlight ruby %} 
    /**
* Get absolute path to template
*
* @param string|null $template
* @return string
*/
public function getTemplateFile($template = null)
{
     $params = ['module' => $this->getModuleName()];
     $area = $this->getArea();
     if ($area) {
         $params['area'] = $area;
     }
     return $this->resolver->getTemplateFileName($template ?: $this->getTemplate(), $params);
}
{% endhighlight %}

You should be able to see we are using a `_templateFileResolution` object which is set in the constructor and is an instance of the following class `Magento\Framework\View\Design\FileResolution\Fallback\TemplateFile` which in turn extends `File`.

Look at `File::getFile()`:

{% highlight ruby %}
    /**
 * Get existing file name, using fallback mechanism
 *
 * @param string $area
 * @param ThemeInterface $themeModel
 * @param string $file
 * @param string|null $module
 * @return string|bool
 */
public function getFile($area, ThemeInterface $themeModel, $file, $module = null)
{
    return $this->resolver->resolve($this->getFallbackType(), $file, $area, $themeModel, null, $module);
}
{% endhighlight %}

This method is using the `resolver` object to decide as to which is loaded in using DI, in default case we will have `Magento\Framework\View\Design\FileResolution\Fallback\Resolver\Simple` loaded.

This class uses the `RulePool` object and decides which file to use:

{% highlight ruby %}
    $path = $this->resolveFile($this->rulePool->getRule($type), $file, $params);
{% endhighlight %}

The `getRule()` method is a wrapper around the following function with some conditionals (Not important at the moment): 

{% highlight ruby %}
    /**
 * Retrieve newly created fallback rule for dynamic view files
 *
 * @return RuleInterface
 */
protected function createFileRule()
{
    return $this->modularSwitchFactory->create(
        ['ruleNonModular' => $this->themeFactory->create(
            ['rule' => $this->simpleFactory->create(['pattern' => "<theme_dir>"])]
        ),
        'ruleModular' => new Composite(
            [
                $this->themeFactory->create(
                    ['rule' => $this->simpleFactory->create(['pattern' => "<theme_dir>/<module_name>"])]
                ),
                $this->moduleFactory->create(
                    ['rule' => $this->simpleFactory->create(['pattern' => "<module_dir>/view/<area>"])]
                ),
                $this->moduleFactory->create(
                    ['rule' => $this->simpleFactory->create(['pattern' => "<module_dir>/view/base"])]
                ),
            ]
        )]
    );
}
{% endhighlight %}

You can use the DI to add another rule to the `rulePool` and change the higher-archy here.

##Blocks

Magento 2 still has Blocks which are fairly unique to Magento. A Block is pretty powerful and provides a set of re-usable methods for you're templates. Just like in Magento 1.x Blocks have several types:

- Template
- Text
- ListText
- Messages
- Redirect

All of these block types will extend `AbstractBlock` which has the `toHtml()` method in, this is part of the service contract implemented by the `BlockInterface`.

Containers do not contain any content, they just determine the layout of a page; blocks are the items that have the job of containing content and will be rendered within a container. Blocks can contain blocks, containers can contain blocks, blocks can contain containers and containers can contain containers; don't get it? Just go watch inception.
 
 In all seriousness if you haven't worked with blocks in Magento before, you're probably fairly confused by this explanation.
 
 However blocks are actually a really simple concept. I have taken this image from Alan Kent, i'm sure he won't mind; here is a diagram of blocks overlaid:
 
![Visual block representation](https://alankent.files.wordpress.com/2015/01/term-blocks-content.jpg)
  
Source (Which I really recommend reading): (http://alankent.me/2015/01/24/magento-2-containers-and-blocks/)[Alan Kent - MAGENTO 2 CONTAINERS AND BLOCKS]

Remember I mentioned the `Page::renderPage()` method earlier, this calls the `rootTemplate` block, this template is `root.phtml`:

{% highlight html %}
    <?php
/**
 * Copyright © 2015 Magento. All rights reserved.
 * See COPYING.txt for license details.
 */

// @codingStandardsIgnoreFile

?>
<!doctype html>
<html <?php /* @escapeNotVerified */ echo $htmlAttributes ?>>
    <head <?php /* @escapeNotVerified */ echo $headAttributes ?>>
        <?php /* @escapeNotVerified */ echo $requireJs ?>
        <?php /* @escapeNotVerified */ echo $headContent ?>
        <?php /* @escapeNotVerified */ echo $headAdditional ?>
    </head>
    <body data-container="body" data-mage-init='{"loaderAjax": {}, "loader": { "icon": "<?php /* @escapeNotVerified */ echo $loaderIcon; ?>"}}' <?php /* @escapeNotVerified */ echo $bodyAttributes ?>>
        <?php /* @escapeNotVerified */ echo $layoutContent ?>
    </body>
</html>
{% endhighlight %}

Eeeeew? Where's are these magic variables coming from? Calm down, they are set with the `Magento\Framework\View\Result\Page` class using the `assign()` method, remember we are currently within the output buffering.
  
{% highlight ruby %}
    $this->assign([
      'requireJs' => $requireJs ? $requireJs->toHtml() : null,
      'headContent' => $this->pageConfigRenderer->renderHeadContent(),
      'headAdditional' => $addBlock ? $addBlock->toHtml() : null,
      'htmlAttributes' => $this->pageConfigRenderer->renderElementAttributes($config::ELEMENT_TYPE_HTML),
      'headAttributes' => $this->pageConfigRenderer->renderElementAttributes($config::ELEMENT_TYPE_HEAD),
      'bodyAttributes' => $this->pageConfigRenderer->renderElementAttributes($config::ELEMENT_TYPE_BODY),
      'loaderIcon' => $this->getViewFileUrl('images/loader-2.gif'),
  ]);

$output = $this->getLayout()->getOutput();
$this->assign('layoutContent', $output);
{% endhighlight %}

This post is mainly aimed at Magento 1.x developers, so I won't go into any more detail about blocks, however Alan Kent's post should answer all your questions.

##Design Layout XML Schema

If you have come from Magento 1.x, you should be familiar with layout XML. Layout XML is a fundamental part of rendering in Magento, it helps you control the output of you're templates and what block they should be using.

Layout is pretty much the same as in Magento 1.x, however now we have an exact schema to work with. Also you can add your layout XML within your module `Namespace/Module/view/{area}/layout/some_layout.xml`.

You can also now render attributes within XML:

{% highlight xml %}
    <page>
    <html>
        <attribute name="foo" value="bar" />
    </html>
</page>
{% endhighlight %}

To add assets your in XML by using the following code:

{% highlight xml %}
    <page>
    <head>
        <css src="Namespace_Module:css/style.css" />
        <script src="js/script.js" />
        <link src="Namespace_Module:js/object.js" />
        <remove src="js/script.js" />
    </head>
</page>
{% endhighlight %}

We now have auto completion in our IDE's so writing XML should be a lot less painful.

We now have more elements to work with in XML within our containers, like Ui components!:

{% highlight xml %}
    <page>
    <body>
         <move element="foo" destination="bar" before="-" />
         <ui_component name="foo_bar" component="some_component" />
         
         <referenceBlock name="example_block">
            <!-- updates -->
         </referenceBlock>
         
         <referenceContainer name="example_container">
            <!-- updates -->
         </referenceContainer>
         
         <container name="some_new_container">
            <!-- just like defining a block -->
         </container>
         
    </body>
</page>
{% endhighlight %}

We also bow have a arguments node where we can specify objects or arrays, if two items in the array share the same key Magento will use the second item, (this is seriously cool!):
 
{% highlight xml %}
    <arguments>
     <argument name="ruleLoader" xsi:type="object">Magento\Authorization\Model\Acl\Loader\Rule</argument>
     <argument name="product_configuration_helpers" xsi:type="array">
        item name="bundle" xsi:type="string">Magento\Bundle\Helper\Catalog\Product\Configuration</item>
     </argument>
</arguments>
{% endhighlight %}

Layout handles are the same in Magento 2, however we can now add layout handles in XML `<update handle="new_handle" />` - I can't really think as to why I would need to use that yet except for keeping my code nicely separated. You can also use `Page::addHandle()` to add a layout handle.
