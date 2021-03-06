Django-languages
================
A field for languages as data.

Please note that this app is not for internationalization, which is translation of the display text in a site etc. This app is for the input and handling of a language code as data (thus it is much simpler than internationalization). The app may be used, for example, to record the languages a user speaks.

If a language is in the data within this app, it does not mean Django has translation ability for that language. If the 'language of Ni!' is listed, Django may not be able to translate the 'language of Ni!'. But this app can record that a person can speak the 'language of Ni!' (oh no it can't! Because ISO639-3 doesn't record 'Ni!' as a language, which it plainly is, even if it only has one word). 
 
Limitations
-----------
The app is grounded in `ISO 639-3`_, the (slightly contentious) ISO (more or less) standard. So it only deals in three-letter codes, not locale-like codes. That means, for example, the app can not express the full range of Django translations (as Django contains, for example, translations expressed using locale subtags like 'en-gb', 'en-as').

(on the other hand, 639-3 currently can express 7000+ languages in it's three-letter codes, and is a web standard)

Alternatives
------------
A module called 'django-languages' already exists in the Python Software package index,
https://pypi.python.org/pypi/django-languages/0.1 . I have ignored this module. It is not updated for several years.

The app was based in a app called https://github.com/audiolion/django-language-field . This was what I wanted, a form field, but was lacking several facilities (sort, query, etc.) I didn't know how to fork the project, and have replaced the code. 

Some django-languages facilities are taken from an excellent, long-standing django app, django-countries https://github.com/SmileyChris/django-countries/tree/master/django_countries (though I am some way from full replication). Like django-countries, this app is not Model-based.

There are several language apps (though not as many as 'countries' apps),

The original django-language-field,
    https://github.com/audiolion/django-language-field 

As a template tag django-languageselect,
    https://github.com/RegioHelden/django-languageselect
     
     
Model-based 
~~~~~~~~~~~
django-world-languages
    https://github.com/blag/django-world-languages

django-languages-plus. Works with 'django-countries',
    https://pypi.python.org/pypi/django-languages-plus/1.0.0


Installation/dependencies
--------------------------
Place the code in your environment. No other installation needed.


The app
-------


Language sets
~~~~~~~~~~~~~
There are many language classifications available. Sets that attempt to cover all/most languages are the size of books. I'm tired of apps that do not declare their intent (or bias) on this issue. And it is an issue.


The langbase
~~~~~~~~~~~~
Though the app has no database model, it provides an in-memory language 'base'. This contains much of the data from ISO 639-3. From there, you do a query.


The LanguageChoices class
~~~~~~~~~~~~~~~~~~~~~~~~~~
This class holds a result from the langbase. Of course, because the langbase is a crude in-memory item, the LanguageChoices is not as sharp in it's queries as a database query language. But, for these purposes, it should be enough.

LanguageChoices delivers a set of language pair tuples to a Django field. Within, it contains an attribute 'queryset', which can be queried for the language data held by the choices.

Form some choices, ::

    from django_languages import LanguageChoices
    
    LanguageChoices()

By default this will fail. It needs languages declaring. Form a set of language data, selecting by three-letter code, ::

    lc = LanguageChoices(pk_in=['eng', 'por', 'spn'])
    
See the data these codes have selected, ::

    for l in lc:
        print(l)

Please note that order of presentation is the order of pk_in (and other explicit selectors shown below also order the display according to the order of declaration).

To select only living languages (big list), use the 'type' column in the langbase ::

    lc = LanguageChoices(type_in=['L'])

The `ISO 639-3 table`_ page contains links to notes on the table structure.

There is a twist. 639-3 includes some special codes for 'undefined' or 'not a language' marks. By default, the app excludes them. You can put them back in, ::

    lc = LanguageChoices(special_pk_in=['und'])

appends the und(efined) mark to the queryset.


Presets
+++++++
A few presets have been built for 'pk_in'. All are contentious. But, if you are not contending this issue, why not?

UNITED_NATIONS
    Official languages of the UN. 6 entries.

INTERNET_MOST_CONTENT
    From https://en.wikipedia.org/wiki/Languages_used_on_the_Internet.
    Contentious subject, but a useful near-Europe set. 39 entries.
    
INTERNET_MOST_TRAFFIC
    From a Wikipedia link, a list contending the above and very 
    different (more coverage of languages from Asia). 15 entries.
     
DJANGO_TRANSLATED
    Django translations from django.conf.global_settings, 2017. Not exact; 
    some dialects dropped, and added plain Chinese/English.
    Will reflect areas with computing and Python coding. 78 entries.

MOST_SPEAKERS
    from https://en.wikipedia.org/wiki/List_of_languages_by_number_of_native_speakers. Less contentious list than most, Lack of cultural targets may limit use. Some coverage for neglected areas such as South Asia and Chinese distinctions. 100 entries.

Or make your own.

Other LanguageChoices options
++++++++++++++++++++++++++++++

override
    Change the common name of one of the languages e.g. override = {fra : "Chez nous"} 
     
first
    A trick from 'django-countries'. Pull out some country data and put it first in the list. first_repeat=True (default=False) will repeat that data in the main list.

sort
    (default=True, if sort=False you get the list as in the langbase) Sort the body entries. For more accurate sorting of translated country names, install the optional pyuca_ package for Unicode collation. Not customizable, but better than usual.

reverse
    Sort backwards.
    

Implementation options
~~~~~~~~~~~~~~~~~~~~~~

Templating, a reminder
++++++++++++++++++++++
Django-languages stores language data as the 3-letter code. Another way of saying this is that the data, in some minor way, is serialised (this is not special, this is true for all Django 'choices' fields).

There will be some cases when you wish to display the language code itself. Using the field named as it is below, in a template, something like, ::    
    
    <li>{{ object.lang }}</li>
    
But, more often, you will want to display the common name of the language, ::

    <li>{{ object.get_lang_display }}</li>
    
As choices
++++++++++
LanguageChoices can be used in a Widget, form.Field or Model.field to provide the 'choices' option. For this use, a Model.field is probably most appropriate (languages options subsituting for a set of fixed options). The field is a Charfield and the option max_length will be 3 (for the code), ::

    from django_languages import LanguageField, LanguageChoices

    class EnglishPaper:
       ...
       # provide middle and old english
       LANGUAGE_CHOICES = LanguageChoices(pk_in=['eng', 'enm', 'ang'])
       
       lang = models.CharField(
          blank = True,
          choices=LANGUAGE_CHOICES,
          max_length=3,
          default = 'eng',
          help_text="Primary language of text.",
          )
      
This method has an advantage of adhering to Django APIs, so maintainability. A disadvantage is that the 'choices' interface is not (in this case) flexible for some potential needs.


LanguageField
+++++++++++++++
This is a Model field made to handle LanguageChoices data. Note it is a Model field, not a Form field. It has been customized so that, if automatic form generation is used (admin modules etc.), it creates the required interface in forms also. Setup looks like this, ::

    from django_languages import LanguageField, LanguageChoices, INTERNET_MOST_TRAFFIC

    class InternationalPaper:
        ...
        # provide several international languages
        LANGUAGE_CHOICES = LanguageChoices(pk_in=INTERNET_MOST_TRAFFIC)
       
        lang = LanguageField(
          "language",
          blank = True,
          blank_label = 'Not stated...',
          multiple= True,
          lang_choices=LANGUAGE_CHOICES,
          default = 'eng',
          help_text="Language in text and/or speech.",
          )  
      
Two features of the custom field can be seen in this example. First, the field can be set to 'multiple' using a configuration option. Second, the 'blank' display can be set using the 'blank_label' option (setting the label is not otherwise possible, as this would usually be provided in the 'choice' iterable, but we are using a code-generated LanguageChoices).

LanguageField has a feature which is not evident, it has custom configuration checking to help set up the field.

Also note one annoying feature; 'lang_choices' must be declared, not 'choices'. The field will otherwise throw an error (the code needs an independant reference to the class data).


Options
_______

blank_label
    The blank option will use text defined here (because the coder can not define the choice tuples for this field, this option can revise the 'blank' name).
  
multiple
    Use a multiple selector, for many languages
  
blank=True only works on single selectors/selections ('blank' can work oddly on multiple selectors). Alternatively, enable and promote the special 369-3 code 'und'(undefined). 

'default' and other Model field attributes should work as expected.

      
LanguageRelatedField
++++++++++++++++++++
It seems a loss to have the ISO639-3 data available for selection, but only display the common name of a language. This Model field extends the custom field idea further. It returns 'rich' data, as if from a 'related' Model. Returns from queries, and into templates, are not a simple string that represents the option e.g. 'Arabic'. They are a class 'Language' based in the lines in the langbase. They look like this, ::

    <Language "zho", "zh", "M", "L", "Chinese">]

Which may be of interest in some display or further-code situations.

Like this, in a model definition, ::

    from django_languages import LanguageRelatedField, LanguageChoices, INTERNET_MOST_TRAFFIC
    
    class InternationalPaper:
        ...
        # provide several international languages
        LANGUAGE_CHOICES = LanguageChoices(pk_in=INTERNET_MOST_TRAFFIC)
        
        lang = LanguageRelatedField(
            "language",
            blank_label = 'Not stated...',
            multiple= False,
            lang_choices=LANGUAGE_CHOICES,
            default = 'fra',
            help_text="(main) Language of the text.",
        )

One issue with this field is that these full-class returns stringify and may serialize in odd ways. The stock form serialisation is assured, but the code can not account for how the returned classes may behave in other contexts. So, if you would like to display 2-letter codes, or 'dead language' icons, or other language detail, then use LanguageRelatedField. For the simple storage of a language code, prefer LanguageField.


Getting and setting LanguageRelatedField
________________________________________
The field coerces the three-letter code held in the database into a full Language class. The returned class instance contains the row data from the langbase. Assume TextModel has a LanguageField 'lang', ::

    >>> o = TextModel.objects.get(pk=1)
    >>> o.lang
    <Language "ara", "ar", "I", "L", "Arabic">
    >>> o.lang.name
    "Arabic"

You can allocate by Language class, or three-letter code ::

    >>> o.lang = 'fra'
    >>> o.lang
    <Language "fra", "fr", "I", "L", "French">


Options
_______
Same as LanguageField.


Templating
__________
The return is a full object. To display the common name of the language, ::

    <li>{{ object.name }}</li>
    
To get the 'type' code, ::

    <li>{{ object.tpe }}</li>
    
etc.

    
.. _ISO 639-3: http://www-01.sil.org/iso639-3/
.. _ISO 639-3 table: http://www-01.sil.org/iso639-3/codes.asp
.. _pyuca: https://pypi.python.org/pypi/pyuca/

