## Database Decision

After extensive thought and uncountable gpt prompts, we were finally able to create what we believe is the most suitable database structure for our NoSQL Firestore database.

Since footballing data is extremely complicated and highly related, a NoSQL database possibly wasn't even the right option. However, given the following points, I believe it was and extremely suitable approach.

1. While footballing data is highly coupled, it doesn't mean we cannot establish relationships using the NoSQL db. Having field references on objects (eg. A Club ID on a Player object), would allow us to overcome relational contraints and proper operational handling would allow us to maintain data integrity.
2. Our queries were not extremely complex and often consisted of only seasonal/league data. This allowed us to leverage NoSQL's rapid indexing properties alongside sharding to be able to host a highly scalable and quick database with large amounts of data.
3. Cost was perhaps our biggest concern. After seeing the eye-watering price of hosting SQL instances, we were extremely happy to see the low-cost NoSQL alternative (Firestore). Additionally, enabling a highly coupled footballing architecture using NoSQL was part of a very alluring challenge that we wanted to take up.

## Database Structure

Following NoSQL best practices, we wanted to implement as flat a hierarchy as possible. At the same time, we realised the all of our principal objects were highly coupled with the 'season' object (Players, Clubs and Leagues, all have seasonal attributes that change from season to season). Keeping these two in mind, we worked extremely hard to build a multi-layered architecture that has objects on both, a seasonal and a perennial/all-time level. The consequent database looks something like this

- Clubs
- Players
- Leagues
- Seasons
  - Clubs (extended object with season-specific attributes)
  - Players (extended object with season-specific attributes)
  - Leagues
  - Matches
- Data Lake

Another feature of this architecture is that the IDs for the nested seasonal and the base level perennial instances are the same. This ensures that the routers and API usage for the client is as simple as possible.

## Limitation and Workarounds

There were several limitations with our NoSQL implementation. These mainly involved maintaining "foriegn key" relationships on objects. That is how could we ensure the integrity of references when creating, updating and deleting objects? To overcome this we did the following.

1. We extensively harnessed FastAPI's usage of dependency injection to ensure before our DAOs created objects that certain properties existed. For example, before adding a Club to a League's standing, we ensure that the Club indeed does exist in the League.
2. This implementation is extended such that the dependency functions that create objects, verify their existence, add relational attributes, etc. are completely unaware of the underlying schemas. The DAOs need to be aware of the schemas and call the relevant functions accordingly. This ensures that if in the future the schemas change, the dependency functions couldn't care less and only the DAOs would have to be potentially updated to call the right functions.
3. The schemas are implemented keeping the NoSQL structure in mind. They are highly extensible and reuse a lot of code. A lot of data normalisation (such as stringifying UUIDs, making Club codes upper case, etc.) has been implemented in the schemas. We know that our NoSQL db will never complain about constraints, but should the payload break the rules, our schemas certainly will.

## Archive and Future

Backup is currently done manually to a Cloud Storage Bucket. Need to write a cloud function to do this once a month ideally in the future.

The database seems to be very low latency. Once the app has started, we get realtime data almost instantly. However, we don't have too much data right now and after scraping, we need to ensure that we maintain our speed levels.

Things to implement in the future:

1. Data migration with Firestore can be problematic. If we change an attribute in the schemas, we can write cloud functions to update the relevant attribute to avoid a bad data state. However, if we encounter such an error, we should add exception handling in the DAOs to deal with it.
2. More cloud functions to run scheduled jobs to pick up data and push it into our db on a weekly basis after matches have been played every week.
3. The DAOs have not been entirely decoupled from the dependencies yet. More functions and refactoring is needed to ensure the DAO primarily responsible for calling relevant dependency functions.
4. Extensive validation. Our validation is very limited right now and only ensures basic relational integrity. We want to reach a point where the DAOs do the least amount of unexpected things but at the same time the data is airtight.
