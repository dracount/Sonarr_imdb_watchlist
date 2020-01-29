# Sonarr_imdb_watchlist
add IMDB watchlist to Sonarr

Using sonar_imdb_watchlist_api_v3

You need the following to enter in the script:
tvdb api - requires a tvdb account

imdb watchlists ls number - The IMDB watchlist requires the ls# which you can get by going to the watchlist and going EDIT which will place something like this in the url: https://www.imdb.com/list/ls099999999/edit?ref_=wl_edt_pwr. Get the ls099999999 number in there and use that to replace the imdb list numbers in the file.

sonarr api key - get from sonarr

sonarr base url - wherever you are hosting this - IP:PORT

This has been tested using nginx reverse proxy. A new version is in the works for local machine hosting.
