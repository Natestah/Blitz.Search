# Blitz.Search

Blitz.Search handles Searching / Serialization of search results for Blitz Search.


# Basic Example of searching file from Search.Test

```csharp
        //Create a simple AND Query from a word
        BlitzAndQuery.QueryMatches("word", out BlitzAndQuery andQuery);

        //Create basic parameters with the above search term as the text search parameter.
        var parameters =  new SearchTaskParameters(andQuery);
        
        //Search the file
        var fileSearcher = new SearchFileContents(fileName, parameters, new CancelationTokenSource());
        fileSearcher.DoSearchFind(out var results);
        
        //Print the results.
        foreach( var result in results )
        {
            Console.WriteLine( result.CapturedContents )
        }
```

# Message Processing

Blitz.Search takes care of Persistence of Search Messages categorizing them by Process->Instance ID and Task management, with the intent of being a locally hosted service someday, if that ever makes sense to do.

Currently it only has an InProcessSearch Handler, which has an internal COPY of the request/reponse to help guard against object updates and interactions.

# Serialization

Blitz.Search uses MessagePack for serialization, this is used to cache Parsed Information to disk for Speedy Recall of prior searches.  

This is typically handled by the UI layers which would store information about which folders are being searched.

# Caches

Upon initial submit of this, Caches are stored to an appdata folder defined in SearchExtensionCache.CacheFolder 

Which points to "%appdata%\NathanSilvers\CacheContents"

# How the 'Indexing' works

the indexing is proprietary here and works to create a per-extension breakdown of unique words.  Then a per file breakdown of the indexes of those unique words.

At the top level we decide which "word index" to search based on matching BlitzAndQuery against the global - per extension list of words.  This is handled at a high level per file, We simply compare predetermined ints for the file to decide if we even need to open the file for searching. 

instead of storing strings per file, which can lead to duplicates in memory as well as take a little more time to cache recall themselves we store a list of ints that represents their position in the Global unique words.  This information is stored per type With the Super fast MessagePack serialization.

Additionally, Blitz.Search Optimizes Unique word storage, because many Unique Identifiers use Hex to describe them, if the character range of the entire word falls beneath the hex word, it is excluded from persistent storage, it is also excluded when users types those words,  an entire query of sub-hex range words would ultimately perform slower, but I have found this to be rare enough to go unnoticed and helps memory use in general.

