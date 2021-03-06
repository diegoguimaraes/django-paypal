Using PayPal Payments Pro (WPP)
===============================

PayPal Payments Pro (or "Website Payments Pro") is a more awesome version of
PayPal that lets you accept payments on your site. This is now documented by
PayPal as a `Classic API
<https://developer.paypal.com/webapps/developer/docs/classic/products/>`_ and
should not be confused with the "PayPal Payments Pro (Payflow Edition)" which is
a newer API.

The PayPal Payments Pro solution reuses code from `paypal.standard` so you'll
need to include both apps. django-paypal makes the whole process incredibly easy
to use through the provided ``PayPalPro`` class.

1. Obtain PayPal Pro API credentials: login to PayPal, click *My Account*,
   *Profile*, *Request API credentials*, *Set up PayPal API credentials and
   permissions*, *View API Signature*.

2. Edit ``settings.py`` and add  ``paypal.standard`` and ``paypal.pro`` to your
   ``INSTALLED_APPS`` and put in your PayPal Pro API credentials.

   .. code-block:: python

       INSTALLED_APPS = [
           # ..
           'paypal.standard',
           'paypal.pro',
       ]
       PAYPAL_TEST = True
       PAYPAL_WPP_USER = "???"
       PAYPAL_WPP_PASSWORD = "???"
       PAYPAL_WPP_SIGNATURE = "???"

3. :doc:`/updatedb`

4. Write a wrapper view for ``paypal.pro.views.PayPalPro``:


   In views.py:

   .. code-block:: python

       from paypal.pro.views import PayPalPro

       def buy_my_item(request):
           item = {"amt": "10.00",             # amount to charge for item
                   "inv": "inventory",         # unique tracking variable paypal
                   "custom": "tracking",       # custom tracking variable for you
                   "cancelurl": "http://...",  # Express checkout cancel url
                   "returnurl": "http://..."}  # Express checkout return url

           kw = {"item": item,                            # what you're selling
                 "payment_template": "payment.html",      # template name for payment
                 "confirm_template": "confirmation.html", # template name for confirmation
                 "success_url": "/success/"}              # redirect location after success

           ppp = PayPalPro(**kw)
           return ppp(request)

5. Create templates for payment and confirmation. By default both templates are
   populated with the context variable ``form`` which contains either a
   ``PaymentForm`` or a ``Confirmation`` form.


   payment.html:

   .. code-block:: html

       <h1>Show me the money</h1>
       <form method="post" action="">
         {{ form }}
         <input type="submit" value="Pay Up">
       </form>

   confirmation.html:

   .. code-block:: html

       <!-- confirmation.html -->
       <h1>Are you sure you want to buy this thing?</h1>
       <form method="post" action="">
         {{ form }}
         <input type="submit" value="Yes I Yams">
       </form>

6. Add your view to ``urls.py``, and add the IPN endpoint to receive callbacks
   from PayPal:

   .. code-block:: python

       urlpatterns = ('',
           ...
           (r'^payment-url/$', 'myproject.views.buy_my_item')
           (r'^some/obscure/name/', include('paypal.standard.ipn.urls')),
       )

7. Connect to the provided signals in ``paypal.pro.signals`` and have them do something useful:

   * ``payment_was_successful``
   * ``payment_was_flagged``


8. Profit.

Alternatively, if you want to get down to the nitty gritty and perform some
more advanced operations with Payments Pro, use the :class:`paypal.pro.helpers.PayPalWPP` class directly.
