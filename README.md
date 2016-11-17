# Deltafeeds - fast delta and deletion files for CSV and JSON dumps

python programs that extract changes and deletions from raw CSV or JSON data dumps. 

You typically need this to more efficiently realize a mirror database or system integration, 
but the master / source system is not able to provide clean delta / diff exports (or not with deletion information). 

TODO: migrate TODOs from code to issues and realize. 

## System Requirements

 * python. e.g. on Mac: `brew install python`
 * JSON feeds only: 
   * yajl native iterative json library (faster): `brew install yajl`
   * some python modules: `pip install ijson cffi mmh3 jsonpath-rw`

## Usage

'newfile' is the path to the new full dump feed and 'lastfingerprintsfile' is the path to the last 
fingerprints file generated by this program.

If the last fingerprints file is given, the program generates:
 * a CSV or JSON that contains only the changes since then is generated, postfixing 
   `.changes.csv` or ´changes.json´ respectively to the `newfile` name. 
 * a JSON file that contains the 

### CSV
`deltacsv newfile idColumn [lastfingerprintsfile]`

'idColumn' is the name or 0-index of the CSV column that contains the reliable ID of the lines. CSVs need a header row. 

### JSON:
`deltajson newfile entriesProperty idJsonPath [lastfingerprintsfile]`
 
'entriesProperty' is the name of the property in the JSON that contains the array of entry objects to be processed and
analyzed for changes. It's typically in the root object, but does not need to (if nested, there should be no other ones
with that name).

'idJsonPath' is a JSONPath to the value _inside_ the entry objects that represents the ID

## Behavior

 * Order and all data in the orginal CSV or JSON are preserved as far as supported by the semantics of the format.  
 * Formatting of original CSV or JSON is not preserved. 
 * Duplicates with same ID value cause a warning, but no error. Only the first occurrence of an ID is processed at all.
 * Assumes a header row in the CSV. 
 * Unsupported CSV formats need to be converted in a separate step, e.g. using csvkit's `csvformat` command.
 * The delta JSON generated has one object per line, making it suitable for "poor man's incremental JSON parsing"
 
## Performance

On a 2012 Macbook Pro:

A 460 MB CSV with 1.2 Million CSV lines is processed in under a minute. 
RAM usage currently peaking in the 300 MBs. 
TODO RAM needs to be debugged as it should be much less since the CSV is supposed to be streamed. 

A ca 100MB JSON file with 50,000 entries is processed in ca. 60 seconds. RAM peak 30MB. 
