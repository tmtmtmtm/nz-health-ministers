Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the [Minister of Health](https://www.wikidata.org/wiki/Q6866167)
looks mostly good, except it wasn't connected (in either direction) to
the [Ministry of Health](https://www.wikidata.org/wiki/Q16933991), so I
fixed that.

Step 2: Tracking page
=====================

Initial PositionHolderHistory list set up at https://www.wikidata.org/w/index.php?title=Talk:Q6866167&oldid=1232867190

Current status: only knows three historic officeholders, one of which
is missing the succession fields.

Step 3: Set up the metadata
===========================

The first step now in a new repo is always to edit [add_P39.js script](add_P39.js)
to set up the Item ID and source URL.

Step 4: Scrape
==============
Comparison/source = [Minister of Health (New Zealand)](https://en.wikipedia.org/wiki/Minister_of_Health_(New_Zealand))

    wd ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Scraped cleanly on first pass.

Step 5: Get local copy of Wikidata information
==============================================

Again, we can now get the argument to this from the JSON, so call it as:

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json


Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

40 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/b511c61835082/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

5 additions -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/a2d8ea591d929/

Also one mismatch, where Wikidata current lists Annette King as having
replaced Bill English, but Wikipedia suggests that should be Wyatt
Creech instead. Her infobox agrees with the Creech suggestion, so I'm
going to accept that:

    wd uq 'Q4769089$325CA755-0F0B-4C9B-8149-DD2850F844A4' P1365 Q4384608 Q8039486

Step 8: Refresh the Tracking Page
=================================

Those updates give us a final https://www.wikidata.org/w/index.php?title=Talk:Q6866167&oldid=1232884105

