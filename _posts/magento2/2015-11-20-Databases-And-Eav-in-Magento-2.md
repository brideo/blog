---
layout: post
title: Databases & EAV in Magento 2
categories: magento2
---

##Database Overview
 
Magento 1.x'ers will be glad to hear the Magento 2 databases haven't changed all that much, apart from a few enhancements. We still have `Models`, `Resource Models` and `Resource Collections` just like before and they all serve the same purpose. I'm not going to go into the purpose of each one, but if you're interested please check my notes on [Models, resource models, and collections in Magento 1](http://brideo.co.uk/magento-certification-notes/working-with-databases-in-magento/Models,-Resource-Models,-and-Collections/).

Magento's PDO class is now `Magento\Framework\Db\Adapter\Pdo\Mysql` which extends `\Zend_Db_Adapter_Pdo_Mysql`. 

###Model

To create a Model in Magento 2 you should extend `\Magento\Framework\Model\AbstractModel`, like before to assign a Resource Model you use a `_init()` method in your `_construct()`:

{% highlight ruby %}
    protected function _construct()
{
    $this->_init('\Magento\Framework\Model\ResourceModel\Db\AbstractDb');
}
{% endhighlight %}
    
Here is what that `_init()` method looks like in `Abstract`:
    
{% highlight ruby %}
    /**
 * Standard model initialization
 *
 * @param string $resourceModel
 * @return void
 */
protected function _init($resourceModel)
{
    $this->_setResourceModel($resourceModel);
    $this->_idFieldName = $this->_getResource()->getIdFieldName();
}
{% endhighlight %}
   
Note, you would not ever assign `\Magento\Framework\Model\ResourceModel\Db\AbstractDb` as your resource model, this is an abstract class which your Resource Model would extend.

###Resource Model

As mentioned your resource model will extend `\Magento\Framework\Model\ResourceModel\Db\AbstractDb`. Again, in Magento 2 just like in Magento 1.x you call an `_init()` method which tells your Resource Model about it's table name and primary key within your `_construct()`:

{% highlight ruby %}
    protected function _construct()
{
    $this->_init('entity_table', 'entity_id')
}
{% endhighlight %}

Here is what that `_init()` method looks like in `AbstractDb`:

{% highlight ruby %}
    /**
 * Standard resource model initialization
 *
 * @param string $mainTable
 * @param string $idFieldName
 * @return void
 */
protected function _init($mainTable, $idFieldName)
{
    $this->_setMainTable($mainTable, $idFieldName);
}
{% endhighlight %}

###Resource Collections

Your Collection class will extend `\Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection`, this also has an `_init()` function which sets the relational Model and Resource Model within your `_contruct()` method:

{% highlight ruby %}
    protection function _construct()
{
    $this->_init('Model', 'resourceModel')
}
{% endhighlight %}

Here is what that `_init()` method looks like in `AbstractCollection`:

{% highlight ruby %}
    /**
 * Standard resource collection initialization
 *
 * @param string $model
 * @param string $resourceModel
 * @return $this
 */
protected function _init($model, $resourceModel)
{
    $this->setModel($model);
    $this->setResourceModel($resourceModel);
    return $this;
}
{% endhighlight %}

The `addFieldToFilter()` methods work the same as in Magento 1.x. 

###More Information

Methods for C.R.U.D operations in Magento are still `load()`, `save()` and `delete()`, we also still have the same event values fired as in Magento 1.x (See article linked above for more information).

Magento 2 does not have 'read' and 'write' adapters like in Magento 1.x, now they have been consolidated into one.

You can call `$model->validateBeforeSave()` which uses a validator factory containing the rules for the model and checks if it can be saved.


##Setup Scripts & Resources

Because Magento is now Composable we have to define our setup script numbers slightly differently, in your `etc/module.xml` you define a `setup_version` attribute on your `module` node:

{% highlight xml %} 
    <module name="Namespace_Module" setup_version="1.0.0" />
{% endhighlight %}

We still have `data` scripts for seeding and, `sql` scripts are now called `schema` scripts and these are for structural database changes.

Once you run a data/schema install script it will never be run again.

The `InstallData` class implements `InstallDataInterface` interface which includes the following method:

{% highlight ruby %}
    public function install(ModuleDataSetupInterface $setup, ModuleContextInterface $context);
{% endhighlight %}

The `InstallSchema` class implements `InstallSchemaInterface` interface which includes the following method:

{% highlight ruby %}
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context);
{% endhighlight %}

You now include all of your Upgrade's in a single class, you can see magento uses PHP's `compare_version()` function to whether to run a segment of code:

{% highlight ruby %}
    if ($context->getVersion()
    && version_compare($context->getVersion(), '1.0.0') < 0
) {
    //Run some code here.
}
{% endhighlight %}

##Entity, Attribute and Value (EAV)

Magento still has EAV, "Woooo", and they're identical conceptually; which means no more learning for Magento 1.x'ers.

Your Models will still extend `\Magento\Framework\Model\AbstractModel`.

Your Resource Models will extend `\Magento\Eav\Model\Entity\AbstractEntity` which implements the `EntityInterface` and `DefaultAttributesProvider` interfaces, extra methods compared to the standard `\Magento\Framework\Model\ResourceModel\Db\AbstractDb` are:

- `getAttribute('code')` - get an attribute from the entity
- `saveAttribute($this, 'code')` - Save an attribute on an entity without going through the whole entities process

The only initialisation difference is within your Resource Model, instead of called your `_init()` method, you use the proper `__construct()` method and declare your connections:

{% highlight ruby %}
    public function __construct(
        \Magento\Eav\Model\Entity\Context $context,
        $data = []
    ) {
        parent::__construct($context, $data);
        $this->setType('entity_type_code');
        $this->setConnection('entity_type_code_read', 'entity_type_code_write');
    }
{% endhighlight %}

To get meta information regarding the EAV attributes we can use the `Magento\Eav\Model\Config` class with the following methods:

- `getAttribute('entity_type_code', 'attribute_code')`
- `getEntityType('entity_type_code')`

As with before, Eav have there own setup classes, `EavSetup`. This class contains the `addAttribute()` and `updateAttribute()` methods.
