# To use in Shopify to limit quantities of products ordered.

The solution can be easily bypassed by anyone who knows how Shopify works: you can 

+ disable JavaScript, or
+ go directly to the checkout using `/checkout`, or 
+ use `<input type="hidden" name="return_to" value="/checkout">` injected to the add to cart form, or 
+ use a [cart permalink](https://docs.shopify.com/manual/configuration/store-customization/page-specific/cart-page/cart-permalinks) to add any amount of items to the cart at any time.

To make this tutorial work for you, you will need to disable Ajax in your theme. Head over to your Customize theme page, look in the Cart Page section, and un-check the box "Enable Ajaxify Cart". If you use the Brooklyn theme, go to your Customize theme page, look in the Products section, and un-check the box "Enable slide-out shopping cart". This solution will not work in the Lookbook theme.

There are Shopify apps that limit quantities purchased: [https://apps.shopify.com/search/query?q=minimum](https://apps.shopify.com/search/query?q=minimum).

## Installation

STEP 1: Upload the file jquery.limit.min.js to your Shopify theme assets on the Template Editor page.

STEP 2: Add the following code to your theme.liquid layout file before the closing `</body>` tag:

```liquid
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.limit();
</script>
```

## Configuration

Configuration parameters are all optional.

You pass an object to `Shopify.Cart.limit()`.

That object is a hash of key/value pairs.

+  `limitQuantity`: a number, can be set to 0, 1, 2, 3 and up. Use 0 if you want to disable purchase. Default if not specified is 1.

+ `limitPer`: that on which the limit is imposed. Use 'product', 'variant' or 'order'. Default if not specified is 'product'.

+ `limitProductHandles`: an array of product handles. If you don't set that parameter, the limit is imposed on every product in your shop. Default is an empty array.

+ `limitSkipCartPage`: a boolean. True if you want the shopper to skip the cart page and go directly to the checkout when he adds something to the cart. Default is false.

## With no configuration object

`Shopify.Cart.limit();` will limit quantities purchased to 1 per product.

## Need something different?

**Example 1**

Limiting to 3 items per product, so you can buy a max of 3 items of the same product, same or different variants of it:

```liquid
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.limit({ limitQuantity: 3, limitPer: 'product' });
</script>
```

**Example 2**

Limiting to 1 item per variant, so customer can buy 1 variant of a product and 1 other variant of same product:

```liquid
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.limit({ limitQuantity: 1, limitPer: 'variant' });
</script>
```

**Example 3**

You have a promo for a product with handle turtles-box and want to limit quantities to 4 per order just for that product:

```liquid
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.limit({ limitQuantity: 4, limitPer: 'product', limitProductHandles: ['turtles-box'] });
</script>
```

**Example 4**

You sell only one product in your shop, and customers will only want to buy 1 of it:

```liquid
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.limit({ limitQuantity: 1, limitPer: 'order', limitSkipCartPage: true });
</script>
```

## Add theme settings to configure this from the Customize theme page

While you're on the Edit HTML/CSS page, scroll all the way down in the left-hand side panel, and, under **Config**, click **settings_schema.json**.

Scroll down to the very bottom of your settings_schema.json file, and add this code before the last square bracket `]` and after the last parentheses `}`. Make sure to include that first comma `,` since you're modifying a JSON data structure.

```json
  ,
    {
    "name": "Limiting Number of Products Purchased",
    "settings": [
      {
        "type": "select",
        "id": "limit",
        "label": "Limit the number of products purchased?",
        "options": [
          {
            "value": "No limit",
            "label": "No"
          },
          {
            "value": "Enabled for all products",
            "label": "Yes, and impose that limit on all products"
          },
          {
            "value": "Enabled for products listed",
            "label": "Yes, but only for the products listed below"
          }
        ],
        "default": "No limit"
      },
      {
        "type": "select",
        "id": "limit_quantity",
        "label": "If so, what limit do you want to impose?",
        "options": [
          {
            "value": "0",
            "label": "0"
          },
          {
            "value": "1",
            "label": "1"
          },
          {
            "value": "2",
            "label": "2"
          },
          {
            "value": "3",
            "label": "3"
          },
          {
            "value": "4",
            "label": "4"
          }
        ],
        "default": "1"
      },
      {
        "type": "select",
        "id": "limit_per",
        "options": [
          {
            "value": "variant",
            "label": "per variant"
          },
          {
            "value": "product",
            "label": "per product"
          },
          {
            "value": "order",
            "label": "items per order"
          }
        ],
        "default": "product"
      },
      {
        "type": "textarea",
        "id": "limit_product_handles",
        "label": "Enter your comma-separated product handles here:",
        "info": "(Leave blank if you are imposing your limit on all products.)"
      },
      {
        "type": "checkbox",
        "id": "limit_skip_cart_page",
        "label": "Skip the cart page if limit is 1 item per order?",
        "default": true
      }
    ]
  }

```

Click Save.

Before the closing `</body>` tag in your theme.liquid file, add this:

```liquid
{% unless settings.limit == 'No limit' %}
{{ 'jquery.limit.min.js' | asset_url | script_tag }}
<script>
Shopify.Cart.setProductHandle('{{ product.handle }}');
Shopify.Cart.limit( {
  limitPer: '{{ settings.limit_per }}',
  limitQuantity: {{ settings.limit_quantity }}{% if settings.limit == 'Enabled for products listed' %},
  limitProductHandles: jQuery.trim("{{ settings.limit_product_handles }}").split(/[\s,;]+/){% endif %}{% if settings.limit_per == 'cart' and settings.limit_quantity == '1' %},
  limitSkipCartPage: {% if settings.limit_skip_cart_page %}true{% else %}false{% endif %}{% endif %}
  } );
</script>
{% endunless %}
```

Click Save.

There are many different ways to configure the Limiter in your shop. Here's a look at five scenarios.

### First scenario

You want to limit quantities purchased to 3 items per product, so that shoppers can buy a max of 3 items of the same product — same or different variants of it.

In your Customize theme section for limiting, use the following settings:

* **Limit the number of products purchased?** – select <kbd>Yes, and impose that limit on all</kbd>
* **If so what limit do you want to impose?** - select <kbd>3</kbd>, and <kbd>per product</kbd>

![Alt text](https://monosnap.com/file/uD8ofU6519YeWZuSnMPCUnOz9csz46.png)

Example: if you have an Awesome Tee available in Small, Medium and Large, shoppers will be able to purchase 2 X Small plus 1 X Medium of your Awesome Tee (or less).

### Second scenario

You want to limit quantities purchased to 1 item per product, so that shoppers can buy a max of 1 item of any given product.

Example: you sell clothes that come in various sizes and styles. There can be up to 60 variants per product in your shop. You do not want the shopper to buy the same dress in *Small Blue* and *Medium Red*.

In your Customize theme section for limiting, use the following settings:

* **Limit the number of products purchased?** – select <kbd>Yes, and impose that limit on all</kbd>
* **If so what limit do you want to impose?** - select <kbd>1</kbd>, and <kbd>per product</kbd>

![Alt text](https://monosnap.com/file/vzNsyJ21z5oCpC1uwaN15bHo5xVjpa.png)

### Third scenario

You want to limit quantities purchased to 1 item per variant, so that shoppers can buy a max of 1 item of the same product variant.

In your Customize theme section for limiting, use the following settings:

* **Limit the number of products purchased?** – select <kbd>Yes, and impose that limit on all</kbd>
* **If so what limit do you want to impose?** - select <kbd>1</kbd>, and <kbd>per variant</kbd>

![Alt text](https://monosnap.com/file/c3K13CQB4WELDQSuzWbhgPPa9nopxe.png)

Example: if you have an Awesome Book product available in Soft Cover and PDF download, shopper will be able to purchase both the Soft Cover and the PDF, but only one of each.

### Fourth scenario

You have a “deal of the day” product, and its handle is “moon-palace-all-inclusive-golf-spa-resort-cancun-for-2-people”. You want to limit purchases of this deal to 1 per order. You don't want to limit purchases of any other product in your shop.

In your Customize theme section for limiting, use the following settings:

* **Limit the number of products purchased?** – select <kbd>Yes, and impose that limit on all</kbd>
* **If so what limit do you want to impose?** - select <kbd>1</kbd>, and <kbd>per product</kbd>
* In the **Enter your comma-seperated product handles here** section, type in your product handle, e.g. <kbd>moon-palace-all-inclusive-golf-spa-resort-cancun-for-2-people</kbd>

![Alt text](https://monosnap.com/file/hH3a0JmbdSH0fXdGFJrdr2cYycSQNP.png)

### Fifth scenario

You want to limit products purchased to 3 per order. Shoppers can pick any 3 products they want in your store and check out with them — but not more. It could be 3 items of the same product.

In your Customize theme section for limiting, use the following settings:

* **Limit the number of products purchased?** – select <kbd>Yes, and impose that limit on all</kbd>
* **If so what limit do you want to impose?** - select <kbd>3</kbd>, and <kbd>items per order</kbd>

![Alt text](https://monosnap.com/file/93HuTsxhfAq9S14zA2Gljk10URLWdO.png)
