[![Discord](https://img.shields.io/discord/665254494820368395?color=7389D8&label=chat&logo=discord&logoColor=ffffff)](https://vaticle.com/discord)
[![Discussion Forum](https://img.shields.io/discourse/https/forum.vaticle.com/topics.svg)](https://forum.vaticle.com)
[![Stack Overflow](https://img.shields.io/badge/stackoverflow-typedb-796de3.svg)](https://stackoverflow.com/questions/tagged/typedb)
[![Stack Overflow](https://img.shields.io/badge/stackoverflow-typeql-3dce8c.svg)](https://stackoverflow.com/questions/tagged/typeql)

# TypeDB CTI: An Open Source Threat Intelligence Platform with STIX

- [Overview](#overview)
- [STIX](#stix)
- [MITRE ATT&CK Data](#mitre-attck-stix-data)
- [Installation](#installation)
- [Example Queries](#example-queries)
- [Explorer Utility](#explorer-utility)

## Overview

*Watch our first community webinar [here](https://www.youtube.com/watch?v=xuiYorG8-1Q) or read [this blog](https://blog.vaticle.com/introducing-a-knowledge-graph-for-cyber-threat-intelligence-with-typedb-bdb559a92d2a) post for an introduction.*

TypeDB Data - CTI is an open source threat intelligence platform for organisations to store and manage their cyber threat intelligence (CTI) knowledge. It enables threat intel professionals to bring together their disparate CTI information into one database and find new insights about cyber threats.

The benefits of using TypeDB for CTI: 
1. TypeDB enables data to be modelled based on [logical and object-oriented principles](https://docs.vaticle.com/docs/schema/overview). This makes it easy to create complex schemas and ingest disparate and heterogeneous networks of CTI data, through concepts such as type hierarchies, nested relations and n-ary relations.
2. TypeDB's ability to perform [logical inference](https://docs.vaticle.com/docs/schema/rules) during query runtime enables the discovery of new insights from existing CTI data — for example, inferred transitive relations that indicate the attribution of a particular attack pattern to a state-owned entity. 
3. TypeDB enables links between hash values, IP addresses, or indeed any data value that is shared to be made by default, as uniqueness of attribute values is a database guarantee. When attributes are inserted, unique values for any data type are only stored once, and all other uses of that value are connected by relations.

![TypeDB Studio](images/query_0.png)

This repository provides a schema that is based on [STIX2](https://oasis-open.github.io/cti-documentation/), and contains [MITRE ATT&CK](https://github.com/mitre-attack/attack-stix-data) as an example dataset to start exploring this threat intelligence platform. In the future, we plan to incorporate other cyber threat intelligence standards and data sources, in order to create an industry-wide data specification in TypeQL that can be used to ingest any type of threat intel data. 

## STIX

[Structured Threat Information Expression (STIX™)](https://oasis-open.github.io/cti-documentation/) is a language and serialization format used to exchange cyber threat intelligence (CTI).

STIX enables organizations to share CTI with one another in a consistent and machine readable manner, allowing security communities to better understand what computer-based attacks they are most likely to see and to anticipate and/or respond to those attacks faster and more effectively.

STIX is designed to improve many different capabilities, such as collaborative threat analysis, automated threat exchange, automated detection and response, and more.

The data model in TypeDB Data - CTI is currently based on STIX (specifically STIX 2.1), offering a unified and consistent data model for CTI information from an operational to strategic level. This enables the ingestion of heterogeneous CTI data to provide analysts with a single common language to describe the data they work with.  

To learn more about STIX, this [introduction](https://oasis-open.github.io/cti-documentation/stix/walkthrough) and [explanation](https://oasis-open.github.io/cti-documentation/examples/visualized-sdo-relationships) is a good place to start learning how STIX works and why TypeDB Data - CTI uses it. 

An in-depth overview of the how the STIX2 model has been implemented in TypeDB will follow. 

## MITRE ATT&CK STIX Data

[MITRE ATT&CK](https://github.com/mitre-attack/attack-stix-data) is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations. The ATT&CK knowledge base is used as a foundation for the development of specific threat models and methodologies in the private sector, in government, and in the cybersecurity product and service community.

TypeDB Data - CTI includes a migrator to load MITRE ATT&CK STIX and serves as an example datasets to quickly start exploring this threat intelligence database. 

## Installation 

**Prerequesites**: 
- Python >3.6
- [TypeDB Core 2.10.0](https://vaticle.com/download#core)
- [TypeDB Python Client API 2.9.0](https://docs.vaticle.com/docs/client-api/python)
- [TypeDB Studio 2.10.0-alpha-4](https://vaticle.com/download#typedb-studio)

Clone this repo:

```bash 
git clone https://github.com/typedb-osi/typedb-data-cti
```

Set up a virtual environment and install the dependencies:

```bash
cd <path/to/typedb-data-cti>/
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```
Start TypeDB
```bash 
typedb server
```
Start the migrator script

```bash
python migrate.py
```
This will create a new database called `cti`, insert the schema file and ingest the MITRE ATT&CK datasets; it will take under one minutes to complete. 

## Example Queries

Once the data is loaded, these queries can be used to explore the data. 

1. Does the "Restrict File and Directory Permissions" course of action mitigate the "BlackTech" intrusion set, and if so, how?
```
match
$course isa course-of-action, has name "Restrict File and Directory Permissions";
$in isa intrusion-set, has name "BlackTech";  
$mit (mitigating: $course, mitigated: $in) isa mitigation;
```
This query returns a relation of type `inferred-mitigation` between the two entities: 
 
![TypeDB Studio](images/query_3.png)

But the `inferred-mitigation` relation does not actually exist in the database, it was inferred at query runtime by TypeDB's reasoner. By double clicking on the inferred relation, the explanation shows that the `course-of-action` actually mitigates an `attack-pattern` with the name `Indicator Blocking`, which has a `use` relation with the `intrusion-set`.

![TypeDB Studio](images/query_4.png)

However, that `use` relation (between the `intrusion-set` and the `attack-pattern`) is also inferred. Double clicking on it shows that the `attack-pattern` is not directly used by the `intrusion-set`. Instead, it is used by a `malware` called `Waterbear`, which is used by the `intrusion-set`.

![TypeDB Studio](images/query_5.png)


2. What attack patterns are used by the malwares that were used by the intrusion set APT28?
```
match 
$intrusion isa intrusion-set, has name "APT28"; 
$malware isa malware, has name $n1; 
$attack-pattern isa attack-pattern, has name $n2;
$rel1 (used-by: $intrusion, used: $malware) isa use; 
$rel2 (used-by: $malware, used: $attack-pattern) isa use; 
```
This query asks for the entity type `intrusion-set` with name `APT28`. It then looks for all the `malwares` that are connected to this `intrusion-set` through the relation `use`. The query also fetches all the `attack-patterns` that are connected through the relation `use` to these `malwares`.

The full answer returns 207 results. Two of those results can be visualised in TypeDB Studio like this: 

![TypeDB Studio](images/query_2.png)

3. What are the attack patterns used by the malware "FakeSpy"?
```
match 
$malware isa malware, has name "FakeSpy";
$attack-pattern isa attack-pattern, has name $apn;
$use (used-by: $malware, used: $attack-pattern) isa use; 
```

Running this query will return 15 different `attack-patterns`, all of which have a relation of type `use` to the `malware`. This is how it is visualised in TypeDB Studio: 

![TypeDB Studio](images/query_1.png)

## Explorer Utility

TypeDB CTI provides the following Explorer Utility to help analysts attribute threat groups from indicators.

### Attributing TTPs to Threat Groups

The following will list a set of threat groups that have been used in ATT&CK TTP indicators and observed during an intrusion. 

For example, if T1189 and T1068 have been sighted in a campaign, the following command will show which APT groups are using these.

```
python explorer.py --infer_group --ttp T1189 T1068
```
The result is: 

```
INFO:utils.queries:Total links 36 
INFO:utils.queries:Total nodes 34 
INFO:utils.queries:
+-------------------+-------------+
| Group Name        |   TTP count |
+===================+=============+
| APT32             |           2 |
+-------------------+-------------+
| PLATINUM          |           2 |
+-------------------+-------------+
| Threat Group-3390 |           2 |
+-------------------+-------------+
| Turla             |           2 |
+-------------------+-------------+
INFO:utils.queries:Total groups 4

```
If fewer TTPs are supplied, then there's a greater chance to map TTPs to many more threat groups. 
For example, just looking at T1189 results in 24 different threat groups:

```
python explorer.py --infer_group --ttp T1189
```
The same command will also check if a TTP exists in TypeDB CTI. In this example, T1234 doesn't exist:
```
python explorer.py --infer_group --ttp T1234

```
In this case, the Explorer Utility throws the following error: 

```
ERROR:utils.queries:TTP T1234 not in database
```

It is very important to understand the associations.

For example, this query:

```
python explorer.py --infer_group --ttp T1189 T1068
```

Returns 5 groups including APT32.

But when including the general technique T1222:
```
python explorer.py --infer_group --ttp T1189 T1068 T1222
```
No groups are returned. However, if the right sub-technique is specified: 

```
python explorer.py --infer_group --ttp T1189 T1068 T1222.002
```

APT32 is returned as the only possible threat group. 

Basic information about a technique can be obtained with this command: 

```
python explorer.py -get_info -ttp T1548
```

```
INFO:utils.queries:
+-------+-----------------------------------+--------------------------+--------------------------+
| TTP   | name                              | created                  | modified                 |
+=======+===================================+==========================+==========================+
| T1548 | Abuse Elevation Control Mechanism | 2020-01-30T13:58:14.373Z | 2022-05-11T14:00:00.188Z |
+-------+-----------------------------------+--------------------------+--------------------------+
```

Basic information of a sub-technique is returned with:

```
python explorer.py -get_info -ttp T1548.001
```

```
INFO:utils.queries:
+-----------+-------------------+--------------------------+--------------------------+
| TTP       | name              | created                  | modified                 |
+===========+===================+==========================+==========================+
| T1548.001 | Setuid and Setgid | 2020-01-30T14:11:41.212Z | 2022-05-11T14:00:00.188Z |
+-----------+-------------------+--------------------------+--------------------------+
```

### General Statistics of Key Entities
The Explorer Utility also provides the ability to display general information about TypeDB CTI. 

```
python explorer.py --stats
```

This command will list the number of instances for a few key entity types: 

```
INFO:utils.queries:Total Intrusions Sets 138
INFO:utils.queries:Total Attack Patterns 1626
INFO:utils.queries:Total Mitre Techniques 659
INFO:utils.queries:Total Mitre Sub Techniques 767
INFO:utils.queries:Total Malware 577
INFO:utils.queries:Total Tools 75
```

### Unique techniques associated to intrusion sets

This is a very good metric to assess how a TTP can refer to only one Intrusion Set.

```
python explorer.py --ttp_scores --limit 2 --sort as
```

```
INFO:utils.queries:
+-------+--------------------+
| TTP   |   Intrusion counts |
+=======+====================+
| T1218 |                  1 |
+-------+--------------------+
| T1621 |                  1 |
+-------+--------------------+

```

And viceversa which ones are more widely used.

```
python explorer.py --ttp_scores --limit 2 --sort desc
```

```
INFO:utils.queries:
+-------+--------------------+
| TTP   |   Intrusion counts |
+=======+====================+
| T1105 |                 69 |
+-------+--------------------+
| T1027 |                 67 |
+-------+--------------------+
```

## Community
If you need any technical support or want to engage with this community, you can join the **#typedb-cti** channel in the [TypeDB Discord server](https://vaticle.com/discord) or join our [Discussion Forum](https://forum.vaticle.com/).
