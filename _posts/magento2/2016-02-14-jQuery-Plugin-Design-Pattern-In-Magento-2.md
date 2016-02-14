---
layout: post
title: Creating a jQuery plugin in Magento 2
categories: magento2
---

Magento 2 has switched from Prototype JS to jQuery, Magento 2 also now uses RequireJS which is a JavaScript file and module loader to help you manage your dependencies.

This blog post will be going the design pattern for a jQuery plugin. If you want to see how to add the require JS library to your page or any other integration issues I would recommend checking the [Magento docs](http://devdocs.magento.com/guides/v2.0/javascript-dev-guide/javascript/js-resources.html).

##Namespaced jQuery Plugin Design Pattern

The plugin we are going to create is a really simple element color changer, the example is purely for demo purposes to give you an insight into the design pattern.

Here is what out initial template should look, there is no actual business logic in this file yet, it's just declaring our namespace and setting the default color to `blue`.

I have commented each section of the code to help provide some clarity of the internals.

To speed up workflow, I recommend downloading my [PHP Storm Live Template](/magento2/Magento-2-PHP-Storm-Live-Templates/) for a jQuery plugin.

{% highlight javascript %}
    ;require(['jquery'], function ($) {

    /**
     * Declare a namespace if it doesn't already
     * exist.
     */
    if (!$.Brideo) {
        $.Brideo = {};
    }

    /**
     * Colour switching plugin declaration.
     *
     * @param element
     * @param {{color: string}} options
     * @constructor
     */
    $.Brideo.Color = function (element, options) {

        /**
         * This variable is used because it's
         * easier than binding `this`
         *
         * @type {$.Brideo.Color}
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
        base.$element.data("Brideo.Color", base);

        /**
         * Plugin constructor
         */
        base.init = function () {
            base.options = $.extend({}, $.Brideo.Color.defaultOptions, options);

        };

        /**
         * Initialize the plugin.
         */
        base.init();
    };

    /**
     * Default options for the plugin.
     *
     * @type {{color: string}}
     */
    $.Brideo.Color.defaultOptions = {
        color: "blue"
    };

    /**
     * Setup the plugin, this will override the default
     * options with the object passed in instantiation.
     *
     * @constructor
     *
     * @param options {{color: string}}
     */
    $.fn.brideo_Color = function
        (options) {
        return this.each(function () {
            (new $.Brideo.Color(this, options));
        });
    };

});
{% endhighlight %}

Now we have a base structure to work with we can add the logic, I will put the function `setColor()` in my `init()` function so it's fired on initialization.

The final product of our color switcher plugin will look like this:

{% highlight javascript %}
    ;require(['jquery'], function ($) {

    /**
     * Declare a namespace if it doesn't already
     * exist.
     */
    if (!$.Brideo) {
        $.Brideo = {};
    }

    /**
     * Colour switching plugin declaration.
     *
     * @param element
     * @param {{color: string}} options
     * @constructor
     */
    $.Brideo.Color = function (element, options) {

        /**
         * This variable is used because it's
         * easier than binding `this`
         *
         * @type this
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
        base.$element.data("Brideo.Color", base);

        /**
         * Plugin constructor
         */
        base.init = function () {
            base.options = $.extend({}, $.Brideo.Color.defaultOptions, options);
            base.setColor();
        };

        /**
         * Set the background color of the element.
         *
         * @returns {$.Brideo.Color}
         */
        base.setColor = function () {
            base.$element.css("background-color", base.options.color);

            return base;
        };

        /**
         * Initialize the plugin.
         */
        base.init();
    };

    /**
     * Default options for the plugin.
     *
     * @type {{color: string}}
     */
    $.Brideo.Color.defaultOptions = {
        color: "blue"
    };

    /**
     * Setup the plugin, this will override the default
     * options with the object passed in instantiation.
     *
     * @constructor
     *
     * @param options {{color: string}}
        */
    $.fn.brideo_Color = function (options) {
        return this.each(function () {
            (new $.Brideo.Color(this, options));
        });
    };

});
{% endhighlight %}

We can then change a colour of an element by doing something like this:

{% highlight html %}
    <input id="button" style="background-color: black" type="button" value="Some Button" />
<script type="text/javascript">

    require(['jquery'], function ($) {
        $('#button').click(function () {

            $('#button').brideo_Color({
                color: "blue"
            });

        });
    });
</script>
{% endhighlight %}

As mentioned, this code isn't a very good real world example however it should provide you with a good idea into how you can create a jQuery plugin of your own. 
