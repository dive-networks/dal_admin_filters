dal_admin_filters
=================

Django-autocomplete-light filters for django admin.

.. figure:: https://raw.githubusercontent.com/shamanu4/dal_admin_filters/master/shot_01.png
   :alt: Admin filter with Select2 input

   alt text

Requirements
------------

This is extension for django-autocomplete-light so you need to install
and configure it too.

Here will be minimum setup for example.

Refer to http://django-autocomplete-light.readthedocs.io/ for more
detailed instructions.

Installation
------------

-  Install using pip

    .. code::

        pip install django-autocomplete-light dal_admin_filters

-  Update INSTALLED\_APPS. You need too put django-autocomplete-light before admin

    .. code:: python

       INSTALLED_APPS = [
           'dal',
           'dal_select2',
           'dal_admin_filters',
           #
           'django.contrib.admin',
           ... other stuff there ...
       ]

Configuration
-------------

-  Create autocomplete view
-  Let our models look like this

   .. code:: python

       class Country(models.Model):
           name = models.CharField(max_length=100, unique=True)

           def __str__(self):
               return self.name


       class Person(models.Model):
           name = models.CharField(max_length=100, unique=True)
           from_country = models.ForeignKey(Country)

           def __str__(self):
               return self.name

-  Then autocomplete view for country selection will be similar to next

   .. code:: python

       from your_countries_app.models import Country

       class CountryAutocomplete(autocomplete.Select2QuerySetView):
       def get_queryset(self):
           # Don't forget to filter out results depending on the visitor !
           if not self.request.user.is_authenticated():
               return Country.objects.none()

           qs = Country.objects.all()

           if self.q:
               qs = qs.filter(name__istartswith=self.q)

           return qs

-  Register view in urls.py

   .. code:: python

          from your_countries_app.views import CountryAutocomplete

          urlpatterns = [
              url(
                  r'^country-autocomplete/$',
                  CountryAutocomplete.as_view(),
                  name='country-autocomplete',
              ),
              url(r'^admin/', admin.site.urls),
          ]

-  Use filter in your admin.py

   .. code:: python

      from django.contrib import admin
      from your_countries_app.models import Country, Person
      from dal_admin_filters import AutocompleteFilter


      @admin.register(Country)
      class CountryAdmin(admin.ModelAdmin):
          pass


      class CountryFilter(AutocompleteFilter):
          title = 'Country from'                    # filter's title
          parameter_name = 'from_country'           # field name - ForeignKey to Country model
          autocomplete_url = 'country-autocomplete' # url name of Country autocomplete view


      @admin.register(Person)
      class PersonAdmin(admin.ModelAdmin):
          class Media:    # Empty media class is required if you are using autocomplete filter
              pass        # If you know better solution for altering admin.media from filter instance
                          #   - please contact me or make a pull request

          list_filter = [CountryFilter]


If setup is done right, you will see the Select2 widget in admin filter
in Person's changelist view.