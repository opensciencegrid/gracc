# Corrections

GRACC corrections are maintained in an Elasticsearch index (`gracc.corrections-0` as of this writing).
These are used by `gracc-request` during summarization to correct/normalize VO and Project names.

The [gracc-correct](https://github.com/opensciencegrid/gracc-tools/tree/master/gracc-correct) tool 
provides a simple interface to manage name corrections. It is a single Python script that requires 
the `elasticseach` package (`pip install elasticsearch`).

Currently GRACC does not expose a read-write interface to Elasticsearch, so the script must be run 
from a node within the GRACC cluster (typically the head/client node).

*Note that it might take a minute for any changes to be reflected in queries to Elasticsearch.*


## Usage

```
gracc-correct [-h] [--url URL] [--index INDEX]
                                 {project,vo} {list,add,update,delete} ...

positional arguments:
  {project,vo}          Correction type
  {list,add,update,delete}
    list                print existing corrections
    add                 create new correction
    update              update existing correction
    delete              delete existing correction(s)

optional arguments:
  -h, --help            show this help message and exit
  --url URL             Elasticsearch URL
  --index INDEX         Index containing corrections
```

### list command

The `list` command will fetch and print the corrections in JSON format. You can limit the number
of results with the `--size` option (default 1000), or filter the results by providing a JSON document
with fields to match, or an arbitrary Elasticsearch query.

#### Extra options:

```
  --size SIZE    Max number of documents to list.
  --query QUERY  Search query to limit results.
  --doc DOC      Optional JSON document with correction source to match
```

### add command

The `add` command with create a new correction. You can provide a JSON document, or 
`gracc-correct` will prompt you for the fields. If a correction already exists with the
given keys, you'll be given the option to update the correction.

#### Extra options:

```
  --doc DOC      Optional JSON document with correction source to match
```

### update command

The `update` command will update an existing correction. You can provide a JSON document, or 
`gracc-correct` will prompt you for the fields. If a correction doesn't already exist with
the given keys, you'll be given the option to create a new correction.


#### Extra options:

```
  --doc DOC      Optional JSON document with correction source to match
```

### delete command

The `delete` command will delete an existing correction. You can provide a JSON document, a 
correction ID, or `gracc-correct` will prompt you for the fields. It will display the matching 
correction, and prompt for confirmation. This will be repeated if multiple documents match.

#### Extra options:

```
  --doc DOC   Optional JSON document with correction source to delete.
  --id ID     Document id to delete.
```

## Examples

### List VO corrections

```
$ gracc-correct vo list --size 5
18 {"ReportableVOName": "nanohub", "VOName": "/nanohub/Role=NULL/Capability=NULL", "CorrectedVOName": "nanohub"}
19 {"ReportableVOName": "ops", "VOName": "/ops/Role=lcgadmin/Capability=NULL", "CorrectedVOName": "ops"}
20 {"ReportableVOName": "osg", "VOName": "/osg/Role=NULL/Capability=NULL", "CorrectedVOName": "osg"}
21 {"ReportableVOName": "osgedu", "VOName": "/osgedu/Role=NULL/Capability=NULL", "CorrectedVOName": "osgedu"}
22 {"ReportableVOName": "star", "VOName": "/star/Role=NULL/Capability=NULL", "CorrectedVOName": "star"}
```

### List Project corrections with query

```
$ gracc-correct project list --query osg
7 {"CorrectedProjectName": "OSG-Staff", "ProjectName": "OSG-Staff"}
24 {"CorrectedProjectName": "OSG-STAFF", "ProjectName": "OSG-STAFF"}
200 {"CorrectedProjectName": "SWC-OSG-UC14", "ProjectName": "SWC-OSG-UC14"}
209 {"CorrectedProjectName": "SWC-OSG-IU15", "ProjectName": "SWC-OSG-IU15"}
104 {"CorrectedProjectName": "osg", "ProjectName": "osg"}
448 {"CorrectedProjectName": "OSG Staff", "ProjectName": "OSG Staff"}
51 {"CorrectedProjectName": "ConnectTrain", "ProjectName": "OSG-Connect-test"}
52 {"CorrectedProjectName": "ConnectTrain", "ProjectName": "OSG-Connect"}
43 {"CorrectedProjectName": "SNOplus", "ProjectName": "OSG-PHY00101"}
```

### List VO corrections matching document

```
$ gracc-correct vo list --doc '{"ReportableVOName":"osg"}' --size 5
20 {"ReportableVOName": "osg", "VOName": "/osg/Role=NULL/Capability=NULL", "CorrectedVOName": "osg"}
495 {"ReportableVOName": "osg", "VOName": "/osg/Role=pilot/Capability=NULL", "CorrectedVOName": "osg"}
920 {"ReportableVOName": "osg", "VOName": "/osg/LocalGroup=external", "CorrectedVOName": "osg"}
1041 {"ReportableVOName": "osg", "VOName": "/osg/LocalGroup=marksant", "CorrectedVOName": "osg"}
1042 {"ReportableVOName": "osg", "VOName": "/osg/Snowmass/Role=snowmassadmin/Capability=NULL", "CorrectedVOName": "osg"}
```

### Add new VO correction interactively

```
$ gracc-correct vo add
Field(s) to correct:
    VOName: example
    ReportableVOName: example
Corrected VOName: osg
Correction created. id: AVyOq_xvTIq8btIx9sGM

$ gracc-correct vo list --query '_id:AVyOq_xvTIq8btIx9sGM'
AVyOq_xvTIq8btIx9sGM {"VOName": "example", "ReportableVOName": "example", "CorrectedVOName": "osg"}
```

### Add new VO correction with doc

```
$ gracc-correct vo add --doc '{"VOName":"example2","ReportableVOName":"example","CorrectedVOName":"osg"}'
Correction created. id: AVyOr3nYTIq8btIx9sGN

$ gracc-correct vo list --query '_id:AVyOr3nYTIq8btIx9sGN'
AVyOr3nYTIq8btIx9sGN {"VOName": "example2", "ReportableVOName": "example", "CorrectedVOName": "osg"}
```

### Update VO correction interactively

```
$ gracc-correct vo update
Field(s) to correct:
    VOName: example2
    ReportableVOName: example
Corrected VOName: fermilab
Correction AVyOr3nYTIq8btIx9sGN updated.

$ gracc-correct vo list --query '_id:AVyOr3nYTIq8btIx9sGN'
AVyOr3nYTIq8btIx9sGN {"VOName": "example2", "ReportableVOName": "example", "CorrectedVOName": "fermilab"}
```

Note: it might take a minute for the update to be reflected in the query.

### Delete correction interactively

```
$ gracc-correct vo delete
Field(s) to correct:
    VOName: example
    ReportableVOName: example
Corrected VOName: osg
{u'VOName': u'example', u'ReportableVOName': u'example', u'CorrectedVOName': u'osg'}
Delete record? (Y/N) y
Correction AVyOq_xvTIq8btIx9sGM deleted.
```

### Delete correction with doc

```
$ gracc-correct vo delete --doc '{"VOName": "example2", "ReportableVOName": "example", "CorrectedVOName": "fermilab"}'
{u'VOName': u'example2', u'ReportableVOName': u'example', u'CorrectedVOName': u'fermilab'}
Delete record? (Y/N) y
Correction AVyOr3nYTIq8btIx9sGN deleted.
```

### Delete correction with id

```
$ gracc-correct vo delete --id AVyOwmxFTIq8btIx9sGP
{u'VOName': u'example', u'ReportableVOName': u'example', u'CorrectedVOName': u'osg'}
Delete record? (Y/N) y
Correction AVyOwmxFTIq8btIx9sGP deleted.
```
