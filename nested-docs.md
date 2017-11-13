# Nested Documents in Solr

Solr supports nested documents as of Solr 4.5.  Limitations prior to Solr 5.x is the lack of faceting at the parent and child level. However, general queries against parent/child work fine.

The Solr structure is flat, and hence nested documents are stored contiguously with the parent document. Note, this means you'll need to update an entire document including all its nested documents even if the change is for a single field.

All the examples are using straight Apache Solr, local filesystem (not solrcloud), and with no security enabled.

## Solr 4.10 Example

### Install Solr

```
wget http://archive.apache.org/dist/lucene/solr/4.10.4/solr-4.10.4.tgz
tar zxvf solr-4.10.4.tgz
```

or for Solr 7

```
wget http://apache.mirror.vexxhost.com/lucene/solr/7.1.0/solr-7.1.0.tgz
tar zxvf solr-7.1.0.tgz
```

### Use the default schema

Solr ships with a default schema that can be re-used for testing purposes:

```
<solr version>/example/solr/collection1/conf/schema.xml
```

To leverage this schema, just make sure any document you send to solr consists of field names that exist in schema.xml.   Note, fields with the `_s` suffix automatically get treated as string fields.

### Start the Solr instance

```
cd <solr version>/example
java -jar start.jar
```

### Delete any existing Documents

There should not be any documents in a scratch install, but keep this command handy

 curl http://localhost:8983/solr/update?commit=true -H "Content-Type: text/xml" --data-binary '<delete><query>*:*</query></delete>'

### Load documents into the collection

Note:

`id` is just an incrementing field, we're incrementing for any field we add including nested/child Documents
`type_s` is a field that denotes which document is the parent. We use this field to find the parent document for any matching child records. Note how child records do not possess this field.


```
curl http://localhost:8983/solr/collection1/update?commit=true -H "Content-Type: text/xml" --data-binary '<add>
   <doc>
     <field name="id">10</field>
     <field name="type_s">parent</field>
     <doc>
       <field name="id">11</field>
       <field name="fname_s">John</field>
       <field name="lname_s">Smith</field>
     </doc>
     <doc>
       <field name="id">12</field>
       <field name="fname_s">Joe</field>
       <field name="lname_s">Blow</field>
     </doc>
     <doc>
       <field name="id">13</field>
       <field name="strno_s">123</field>
       <field name="strname_s">Some Drive</field>
       <field name="strcity_s">Some City</field>
     </doc>
     <doc>
       <field name="id">14</field>
       <field name="strno_s">456</field>
       <field name="strname_s">Another Drive</field>
       <field name="strcity_s">Another City</field>
     </doc>
   </doc>
   <doc>
     <field name="id">15</field>
     <field name="type_s">parent</field>
     <doc>
       <field name="id">16</field>
       <field name="fname_s">Jane</field>
       <field name="lname_s">Doe</field>
     </doc>
     <doc>
       <field name="id">17</field>
       <field name="fname_s">Vicky</field>
       <field name="lname_s">Smith</field>
     </doc>
     <doc>
       <field name="id">18</field>
       <field name="strno_s">321</field>
       <field name="strname_s">Contact Crescent</field>
       <field name="strcity_s">York</field>
     </doc>
     <doc>
       <field name="id">19</field>
       <field name="strno_s">543</field>
       <field name="strname_s">Bleeker Street</field>
       <field name="strcity_s">Athens</field>
     </doc>
   </doc>
 </add>'
```

### Lets run some queries

We're going to run some curl commands to execute the queries. Note that you need to convert special characters.

```
<space>     %20
!           %21
+           %2B
{           %7B
}           %7D
```
Search for any hits where fname_s = John or lname_s = Smith.
Note, this query will return the child documents, not the parent.  There's matching child documents for John Smith and Vicky Smith

```
curl "http://localhost:8983/solr/collection1/select?&indent=true&q=fname_s:John%20OR%20lname_s:Smith"
```

Let's modify the query to return the parent document of any matching child Documents

```
curl "http://localhost:8983/solr/collection1/select?&indent=true&q=%7B%21parent%20which=%27type_s:parent%27%7Dfname_s:John%20OR%20lname_s:Smith"
```

Run a query that returns the Parent document whose Child record has fname_s = John and lname_s = Smith

```
curl "http://localhost:8983/solr/collection1/select?&indent=true&q=%7B%21parent%20which=%27type_s:parent%27%7D%2Bfname_s:John%20%2Blname_s:Smith"
```
