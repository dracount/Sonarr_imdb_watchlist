import csv
import requests
import time
import json
import urllib
from urllib.request import urlopen
import urllib.request as urllib2
import logging
from datetime import datetime
###########
# globals #
###########
sonarr_base_url = 'http://ip:port/api'
sonarr_api_key = 'api_key'
tvdb_api= 'tvdb_api_key'
imdb_lists = ['https://www.imdb.com/list/ls#######/export','https://www.imdb.com/list/ls########/export']

logging.getLogger('requests').setLevel(logging.WARNING)
logging.getLogger('urllib3').setLevel(logging.WARNING)
logger = logging.getLogger('sonarr_imports')
dt = datetime.now()
dt = dt.strftime('%Y/%m/%d %I:%M:%S %p')
logging.basicConfig(filename='logfile.log',level=logging.INFO, format=FORMAT, datefmt='%m/%d/%Y %I:%M:%S %p')

def get_sonarr_series_ids(sonarr_api_key):
    sonarr_series = sonarr_base_url + '/series?apikey=' + sonarr_api_key
    r = requests.get(sonarr_series)
    r.json()
    sonarr = json.loads(r.text)
    sonarr_series = []
    for series in sonarr:
        for entry in series:
            if str(entry) == 'imdbId':
                sonarr_series.append( str(series[entry]))
    return sonarr_series

def get_imdb_series_ids():
    

    imdb_series = []

    for url in imdb_lists:
        r = requests.get(url)
        r.encoding = 'ascii'
        text = r.iter_lines()

        for rows in text:
            row = str(rows).split(',')
            count = 0
            for item in row:
                count = count + 1
                #print(count,'item = '+str(item))
                if count == 6:
                    title = str(item)
                if count == 2:
                    imdbid = str(item)
                if count == 8:
                    if str(item) == 'tvSeries':
                        imdb_series.append(imdbid)
                        
    return imdb_series

def get_tvdbId_from_imdbId(series_to_add):
    tvdb_series_ids = []
    for series_to_search in series_to_add:
        url = 'http://api.tvmaze.com/lookup/shows?imdb=' + series_to_search
        response = urlopen(url)
        data = json.loads(response.read())
        tvdb_series_ids.append(str(data['externals']['thetvdb']))
    return tvdb_series_ids

def get_token_from_tvdb():
    url = "https://api-beta.thetvdb.com/login"
    headers = { "Content-Type": "application/json", "Accept": "application/json" }
    payload = '{ "apikey": "'+tvdb_api+'" }'
    r = requests.post(url=url, data=payload, headers=headers)
    data = r.json()
    return data['token']
    
def get_series_info(token,tvdbId):
    headers = { "Accept": "application/json", 'Authorization' : 'Bearer '+token, "User-agent": "Mozilla/5.0"  }
    url = "https://api.thetvdb.com/series/"+ tvdbId
    r = requests.get(url, headers=headers)
    data = r.json()
    title = data['data']['seriesName']
    slug = data['data']['slug']
    latest_season = data['data']['season']
    return title,slug,latest_season

def add_series_to_sonarr(sonarr_api_key,tvdbId,title,slug,latest_season):
    
    headers = {
		'Content-Type': 'application/json', 
		'X-Api-Key': sonarr_api_key,
		'Accept':'application/json'
    }

    #sonarr_base_url + /profile?apikey=api_key
    #get profiles here. currently using 6?: HD 720/1080 
    #Sonarr post series
    qualityProfileId = '6'
    rootFolderPath = '/Series/'

    post_data = {
        "monitored": 'true', 
        "tvdbId": tvdbId, 
        "title": title, 
        "titleSlug": slug, 
        "seasonFolder": 'true', 
        "qualityProfileId": qualityProfileId, 
        "rootFolderPath": rootFolderPath, 
        #"addOptions": {
        #    "ignoreEpisodesWithFiles": 'false',
        #    "ignoreEpisodesWithoutFiles": 'false'
        #},
        "seasons": [{
            "monitored" : 'false',
            "seasonNumber" : '0'
        }]
    }

    for x in range(1, int(latest_season)):
        post_data['seasons'].extend( [{'monitored': 'false', 'seasonNumber': str(x)}] )
    post_data['seasons'].extend( [{'monitored': 'true', 'seasonNumber': str(latest_season)}] )
    
    url = sonarr_base_url + '/series'
    logger.info(dt + ' : Adding show: ' + title + ' to Sonarr')
    response = requests.post(url=url, json=post_data, headers=headers)
    if response.ok:
        logger.info(dt + ' : Successfully added ' + title)
    else:
        logger.info(dt + ' : Problem adding '+ title +'. See log for more details')
        logger.info(dt + ' : Response: ' + response.content)

def main():
    logger.info(dt + ' : Checking series on IMDB lists to add to sonarr...')
    imdb_series_ids = get_imdb_series_ids()
    sonarr_series_ids = get_sonarr_series_ids(sonarr_api_key)
    series_to_add = list(set(imdb_series_ids) - set(sonarr_series_ids))
    tvdb_series_ids = get_tvdbId_from_imdbId(series_to_add)
    if len(tvdb_series_ids) != 0:
        token = get_token_from_tvdb()
        logger.info(dt + ' : Adding ' + str(len(tvdb_series_ids)) + ' new series..')
        for tvdbId in tvdb_series_ids:
            title,slug,latest_season = get_series_info(token,tvdbId)
            add_series_to_sonarr(sonarr_api_key,tvdbId,title,slug,latest_season)
    else:
        logger.info(dt + ' : No new series to add.')

if __name__ == '__main__':
    main()
