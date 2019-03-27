# tMongoDBOutput2

This is a custom component for Talend Open Studio, Big Data Edition, 
which allows more complex update operations (e.g. $push, $inc, etc.) in one Talend Component.

It is a copy and extension of the standard Talend component tMongoDBOutput.

TODO: At the moment, it only supports one operation at a time (e.g. $push). Adding a second operation (e.g. $push and $inc) results in a concurrent modification exception.

```
Exception in component tMongoDBOutput2_1_Out (parse_payload)
java.util.ConcurrentModificationException
  at java.util.LinkedHashMap$LinkedHashIterator.nextNode(LinkedHashMap.java:719)
  at java.util.LinkedHashMap$LinkedKeyIterator.next(LinkedHashMap.java:742)
  at local_project.parse_payload_0_1.parse_payload.tMongoDBOutput2_1_OutProcess(parse_payload.java:5291)
  at local_project.parse_payload_0_1.parse_payload$1ThreadXMLField_tMongoDBOutput2_1_In.run(parse_payload.java:2297)
```

Installation
============

Create a user component directory and place both subdirectories,
tMongoDBOutput2 and tMongoDBOutput2, into it. Let Talend Open Studio know about the
user component directory by specifying it in Preferences->Talend. You
will then have the new component tMongoDBOutput2 available in the
palette, along with the existing MongoDB components.

Usage
=====

Create the tMongoDBOutput2 component in your job and choose "Upsert with multiple operators" as the "Action on data".

![Screenshot: tMongoDBOutput2 Basic Settings](https://github.com/ckurze/tMongoDBOutput2/tree/master/images/screen_tMongoDBOutput2_Basic_Settings.png)

In the Advanced Setting tab, tick the checkbox to "Generate JSON Document" and provide names for the "Data node" and "Query node". Unchecking "Create empty element if needed" will avoid null values in your target document (unless you provide data types in the JSON tree definition, as the object is not null anymore -- also needs to be fixed, is an error in the original Talend component).

Hit the "..." button to configure the output structure.

![Screenshot: tMongoDBOutput2 Advanced Settings](https://github.com/ckurze/tMongoDBOutput2/tree/master/images/screen_tMongoDBOutput2_Advanced_Settings.png)

In the JSON tree definition, create two subelements with the name for the query node (this will be the first part of the update) and the data node (this will be the action to be performed).

The screenshot below shows how to execute $push into the details array. The tree will be transformed into the following upsert command (as no $ signs are allowed, it will be added automatically - you have to ensure that the direct children of the data node have to be MongoDB operations that can be prefixed with a $ sign).

![Screenshot: tMongoDBOutput2 JSON Tree](https://github.com/ckurze/tMongoDBOutput2/tree/master/images/screen_tMongoDBOutput2_JSON_Tree.png)

The JSON Tree results in the following update statement:

```
db.collection.upsert(
  {
    rowid: <<rowid>>
  },
  {
    $push: {
      details: {
        channelGroup: <<Value of channelGroup, omitted if null>>,
        channel: <<Value of channel, omitted if null>>,
        cluster: <<Value of cluster, omitted if null>>,
        value: <<Value of value, omitted if null>>
      }
    }
  }
)
```

The sample document below in MongoDB shows that the "details" attribute contains values after the execution of the Talend job. Null values are ignored, i.e. only channelGroup is provided for the first element, for the second element, only channelGroup, cluster, and value are provided. The rest of the fields (e.g. channel) is null in the data stream and therefore ignored.

![Screenshot: tMongoDBOutput2 Sample Document](https://github.com/ckurze/tMongoDBOutput2/tree/master/images/screen_tMongoDBOutput2_Sample_Document.png)

