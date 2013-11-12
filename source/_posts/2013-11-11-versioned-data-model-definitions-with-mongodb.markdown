---
layout: post
title: "A use case for MongoDb: Versioned static data model definitions"
date: 2013-11-11 13:53
comments: true
categories: [MongoDB, Databases]
keywords: mongodb,databases,data modelling,ORM,object relational mapping,entity attribute value,eav,document database,nosql
description: ""
published: true
---

At SplashLab, our CMS application supports custom web forms. Each form has a different set of fields which hold different types of data. Email addresses, multi-select radio buttons, text areas, etc. Some forms only get submitted a couple of times. Some are submitted many thousands of times. The best way to store this data is something we are still working on, but I thought I would share how we are using [MongoDb](http://www.mongodb.org/) to solve this problem currently.

<!-- more -->

## Relational Database Solutions

To store both the form definition and the submitted data in a normal relational database you either need many tables (one for each form), or you need to normalize your data schema, usually in the [Entity-Attribute-Value](http://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model) (EAV) style. (This is a common problem in ecommerce or other catalog applications, where products have arbitrary numbers and types of attributes.)

Creating many tables is not desirable. Constantly modifying your relational database schemas by adding new tables is a pain, takes a significant amount of extra application code (dynamic table names?), and can be difficult to manage thousands of tables on the RDBMS side. Another issue is that you have to alter existing tables if attributes are added or removed (or create another new table for this version of the form!), which could involve losing column data or adding lots of null values to rows.

EAV normalization is a hassle as well ([Magento](http://magento.com/) anyone?). To review, this is where each data type (text, int, double, blob, etc) is stored in it's own table. Text values are held in one table, integers in another table, etc, and all rows are keyed by the Attribute and Entity name they belong to. You need an ungodly number of joins to build the final object from the value tables. Inserting new objects requires breaking them apart and saving each value to the correct table. It's also hard to view the data, since it's spread out in multiple tables, making development and debugging more difficult.

## Document Database Solution

Enter MongoDb. (Or another document database?) We store form definitions in one collection. Each document consists of the name of the form, and a sub-array of each attribute which includes it's name, type, position, validation rules, etc. This form definition can be versioned, so if you update the form you can still decode old submission data. Creating another form (or another version of a form) involves simply inserting another document into the collection.

### Example Form Definition Collection

```
    {
      _id : ObjectId("xxxxx123"),  // Mongo Id
      label : "Form 1",
      name : "form1",  // unique, immutable id, enforced in app
      version : 1,
      fields: {
        0 : {
          name : "name",
          label : "Name",
          type : "text",
          length : 128,
          order : 0
        },
        1 : {
          name : "email",
          label : "Email",
          type : "email",
          length : 256,
          order : 1
        }
      }
    },
    {
      _id : ObjectId("xxxxx123"),
      label : "Form 2",
      name : "form2",
      version : 1,
      fields: {
        0 : {
          name : "name",
          label : "Name",
          type : "text",
          length : 128,
          order : 0
        },
        1 : {
          name : "color",
          label : "Favorite Color",
          type : "radio",
          options : {
            "red" : "Red",
            "green" : "Green",
            "blue" : "Blue"
          },
          order : 1
        },
        2 : {
          name : "state",
          label : "State",
          type : "us_states",
          order : 2
        }
      }
    }
```

Then we have a collection that holds all of the form submissions. Each submission is stored in a single document, with a sub-array of all of the data submitted for each field. No matter how different the data is it can all go in to one collection. But unlike a serialized field you can query the data (so long as you also include the form definition ID in the criteria).

### Example Form Submission Collection

```
    {
      _id : ObjectId("xxxxx456"),
      form_name : "form1",
      form_version : 1,
      values: {
        name : "Testy McTesterson",
        email : "testy@test.com"
      }
    },
    {
      _id : ObjectId("yyyyy456"),
      form_name : "form2",
      form_version : 1,
      values: {
        name : "Tester Testopherson",
        color : "blue",
        state : "CA"
      }
    }
```

In theory, record insertion from form submits is simple. Generate the document from the form (including the form_name and form_version), and save it. Done. But if you need to do server-side validation it requires one database read to retrieve the form definition to get the validation rules.

The one piece of data that a form submission is tied to is the form definition, _but_ these definitions should never be deleted or updated, making each form submission is atomic. In the worst case scenario the web form has already been generated when the form is updated to a new version. Then form is submitted. As long as the version number if submitted along with the form data the new submission will be associated with the old form, but should be no other issues. This means we don't need transactions, which is something that rules out using MongoDb in other use cases.

Retrieval of form submissions is fairly efficient since only one extra read is required to get the form definition out of the database (for the presentation logic, so this is sort of optional actually...), then one more database read queries out all submissions that match that form's name and version (be sure you have Indexed those fields!).

## Caveats and Drawbacks

This works well since the form submissions are never going to be updated. They represent the data collected from the a specific form version, and should never have additional data added or removed. If you did need to update the submissions due to a change in the form definition, you would have to do a full scan of the collection and update each document. MongoDb supports multiple document updates, but it would be slow at "big data" scale (remember the [write-lock](http://stackoverflow.com/questions/17456671/to-what-level-does-mongodb-lock-on-writes-or-what-does-it-mean-by-per-connec)!). But updates to a new version of the form should only affect _new_ submissions, so this issue is avoided.

If you wanted to request _all_ submissions, and _then_ attempted to "JOIN" the form definitions for each one, it would be pretty awful - but that doesn't really make sense to do. Generally you start with the form, and get the submissions for that form, not the other way around.

If you are enforcing unique value validation on individual forms, be sure that you have the proper Indexes set on the form_name field in the submission collection, so a `find()` command to assert that the new value is unique only scans through the submissions for that specific form. With all forms using the same collection, it will get large and indexing is important. If a single form has a huge number of entries it might be necessary to add another index just for that form's unique field, but we have not run in to this yet.

One particularly tricky thing you may want to do is retrieve all of the data from multiple versions of a form, _which do not have exactly the same field definitions_. If you store the entries with the form ID and version, you can easily query them all by just omitting the version number (just like you would when checking for unique field values, etc). The challenge is displaying the data. Since we are not updating existing submissions to reflect form definition changes, some submissions will have missing fields or incorrect types. To handle this, we basically join all of the form's version's schemas together into a mega-schema, and make sure we have null checks and loose type coercions in our presentation code for the submissions. This is definitely more "art" (er, "hack") than science.

One way to help with the issue above is to enforce form definition rules in the application code so that you can't make a drastic change to a field definition, like to start using a text field called "first_name" to store a date. You could have a rule where you can't change field definition types, perhaps, so you have to delete the "first_name" field and create a new "date" field, to avoid data type mis-matching. (Or you could just restrict queries to single form versions.)

Something else to watch out for is that if you lose a form definition (bad backup?), then the submission data for that form looses it's meaning. Or if you accidentally modify the form definition for a version, the submissions for that version start to become incoherent ("What was 'option1' defined as in the form definition? How many options were there?").

Finally (although I haven't done a comparison), I assume this is not as space efficient as normalizing the data into a relational DB since you are duplicating the form name and version in each submission, as well as the field key names. Perhaps MongoDb handles this in some clever way, but I'm not aware of it.

## Conclusion

You could still argue that it would be better to normalize this into a Entity-Attribute-Value style relational store with a de-normalized caching layer on top holding the final JOINed object. There are, after all, relations between each submission's data values and the form's definition. But this has been easier to develop, and so far we have not run in to any show stopping issues.

There are trade-offs with any ORM, and giant pitfalls to look out for whenever you have lots of dynamic schema data like this. Most of our application runs on SQL, but we plan to continue to use MongoDb for this component - at least until we don't. ;)





