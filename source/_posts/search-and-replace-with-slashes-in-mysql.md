---
title: Search and Replace with Slashes in MySQL
date: 2019-04-30 01:29:26
tags:
- Laravel
- MySQL

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1556584444/posts/manhattan-skyline.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1556584444/posts/manhattan-skyline.jpg
---
The project im currently working on makes use of Spaties [Event Projector package](https://github.com/spatie/laravel-event-projector). Recently the team at Spatie upgraded the package to Version 2. 

One of the changes that was made was to separate all storable events from regular events by moving them from the `App\Events` to a new `App\StorableEvents` folder. In moving the new stored events a new namespace of `App\StorableEvents` was created.

Whilst this isn't a mandatory change, all new events classes created get in this new namespace and for the sake of keeping code organised, I wanted to move all of our other storable events to this new folder.
Light work until I wanted to replay my events. What I'd forgotten is that the fully qualified class names are stored on the `stored_events` table that the package uses. 

No big deal I thought - ill just search for `App\Events\%` and replace with `App\StorableEvents\%`, how hard could that be in SQL? Turns out harder than I thought but not impossible...

<!--more-->

I wasn't aware of a search and replace feature in MySQL, and whilst the app is still in development I could have dropped the table and started over fresh.
However I had 100's of rows of good [(vs faker generated)](https://github.com/fzaninotto/Faker) test event data I'd put into the system and didn't fancy starting fresh or updating each row by hand. There had to be a better way to do it. 

Usually when I'm not sure on how to do something in SQL is to reach out to my Brother [Nash](https://twitter.com/trashpants). Not only is he awesome with SQL hes a MySQL certified DBA too. He said he wasn't aware of a search and replace function where you were searching and replacing a substring of a column but to look up the `REPLACE` function. And if that wasn't fruitful he could always write me a quick stored procedure.

Whilst the offer was appreciated I 100000% never want to see or use a stored procedure ever again after having to work with them on an old project for over 10 years! So I set about looking for my magical 1 liner to solve the problem.

So it turns out that after some googling the statement I wanted looked something like this:

```sql
UPDATE table 
SET column_name = REPLACE(column_name, old_string, new_string)
WHERE column_name LIKE(old_string.'%')
```

After plugging updating the query for my use case I had:

```sql
UPDATE stored_events 
SET event_class = REPLACE(event_class, 'App\Events', 'App\StorableEvents') 
WHERE event_class LIKE ('App\Events%');
```

Easy AF. 

Except it wasn't.
 
Because of annoying escape characters in MySQL. The updated query with backslashes escaped looked like:

```sql
UPDATE stored_events 
SET event_class = REPLACE(event_class, 'App\\Events', 'App\\StorableEvents') 
WHERE event_class LIKE ('App\\Events%');
```

Still no luck.

I decided at this point to check if I was selecting anything at all with that WHERE LIKE query.

```sql
SELECT * FROM  stored_events WHERE event_class LIKE ('App\\Events%'); 
```

Nothing.

More googling later and it seemed a comical amount of backslashes were needed to escape a backslash in MySQL.

For the string `App\Events` I needed 6 backslashes?!?!?!?! 6!!!! Good thing I could get away without searching for `App\Events\`!

 ```sql
 SELECT * FROM  stored_events WHERE event_class LIKE ('App\\\\\\Events%'); 
 ```
 
Updating this query would now return results.
 
With the new magic number of 6, it seemed fair to assume that 6 slashes would be needed for all backslashes. Finally I could update the `storable_events` table:
  
```sql
UPDATE stored_events 
SET event_class = REPLACE(event_class, 'App\\\\\\Events', 'App\\\\\\StorableEvents') 
WHERE event_class LIKE ('App\\\\\\Events%');
```

Nope.

Rows found, None changed.

It turns out that within `REPLACE` a backslash only needs to be escaped once, so `\\` or 2 backslashes would suffice.

```sql
UPDATE stored_events 
SET event_class = REPLACE(event_class, 'App\\Events', 'App\\StorableEvents') 
WHERE event_class LIKE ('App\\\\\\Events%');
```

Finally I actually had the query I was looking for, and it worked!

In fairness I spent probably far to long for something I probably could have done in PHP with using Laravel Tinker. That being said its always cool to dig into some of the lesser used day to day features of a technology you use regularly. 

I learned something, and hopefully this time waste will help someone else out down the road at some point too!