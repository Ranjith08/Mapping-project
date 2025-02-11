# Prototyping a map interface for Hindutva Watch
- The HW database is a collection of news articles, tweets, and other sources of information about the activities of Hindutva groups in India.
- I believe it is pertinent to develop a easier way to understand relationships between these groups and their activities geospatially

## Prototype
- The database is scraped from the Hindutva Watch website and stored in a CSV
- ```SpaCy``` is used to extract named locations (NER) from the text
- The lowest denominator location is selected and geocoded. This might be a cause for concern as the geocoding is not always accurate. Choices have to be weighted better
- It is then mapped using ```Maplibre```,

## Roadmap
- [ ] Refine geocoding to be more accurate.
- [ ] Research on event classification and other narrative building elements.    