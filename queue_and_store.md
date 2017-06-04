# Message queue and persistence
Currently we are using Flask-SSE which uses redis for both message queue and storage functionality. Redis:
scales poorly, could come into problems even for a single team,
does not support text-based search,
is primarily in-memory and not robust for long-term persistence.

To chose what to replace redis with we need to consider how our application will scale and how to provide search functionality.

## Architecture scaling
We should probably use a single database for user accounts and avoid the Slack problem of having to sign into every team separately. Particularly important for us as the application is supposed to support collaboration.

The answer is less clear for the message queue/store, should we have one for all teams or one per team?

Single queue/store per team:
+ [+ve] data separation between teams, good for security
+ [+ve] makes more sense if allowing people to run their own servers
+ [+ve] number of channels does not have to scale to all teams
- [-ve] overhead of running instance per team (how significant is this?)
- [-ve] more complex to maintain? (For example how would migration happen?)

Other things to note:
I think Slack uses one datastore per team. They use MySQL…
It probably doesn’t make logging/monitoring any more complex to use one store per team, we can still hook everything up to one log aggregator.

## Search functionality
Apache Lucene coupled with a database (Cassandra for example), or Solr, or elasticsearch.
Note, Elasticsearch and Solr both use Lucene underneath.

Elasticsearch is primarily for search and happens to also function as a data store, it can lose writes…

Looks like scaling is more straightforward using Cassandra-Lucene than Solr, because Solr needs to rebuild it’s index whereas Cassandra-Lucene has an index per node.
Cassandra-Lucene seems the most sensible option to try to me (https://github.com/Stratio/cassandra-lucene-index).
