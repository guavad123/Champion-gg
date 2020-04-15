# Champion-gg
get champion.gg dataframe
Get data from [here](https://champion.gg/)
## Import packages
Import packages I need
```python
import pandas as pd
import matplotlib.pyplot as plt
import requests
import bs4
from bs4 import BeautifulSoup
from tqdm.notebook import tqdm
%matplotlib inline
```
## Get URLs
Use `get_championgg_url()` function to crawl champion urls
```python
def get_championgg_url():
    cham_list = []
    url = "https://champion.gg/"
    html = requests.get(url, timeout=30).text
    html = BeautifulSoup(html,"html.parser")
    cham = html.find(name="div", attrs={"class": "col-md-9 clearfix"})
    for cham_data in cham.find_all(name="div", attrs={"class": "champ-height"}):
        virt = []
        # counterexample-> (NUNU & Willump , jungle) (Dr.Mundo , top) (Dr.Mundo , jungle) 
        cham_name = cham_data.find(name="span", attrs={"class": "champion-name"}).string
        cham_name = cham_name.split("&")[0]
        cham_name = cham_name.replace(" ","").replace(".","")
        virt.append(cham_name)
        # get champion lanes
        for lane in cham_data.find_all(name="a")[1:]:
            cham_lane = lane.string
            cham_lane = cham_lane.replace(" ","")
            cham_lane = cham_lane.replace("\n","")
            virt.append(cham_lane)
        cham_list.append(virt)
    cham_url_list = []
    # get urls for every champion lanes
    for i in range(len(cham_list)):
        for lane in cham_list[i][1:]:
            cham_url=url+"champion/"+str(cham_list[i][0])+"/"+str(lane)
            cham_url_list.append(cham_url)
    return cham_url_list
```
## Get champion data
Use `get_championgg_data()` to crawl champion statistics
```python
def get_championgg_data(url):
    html = requests.get(url, timeout=30).text
    html = BeautifulSoup(html,"html.parser")
    tbody = html.find(name="tbody")
    cham_name = url.split("/")[-2]
    cham_lane = url.split("/")[-1]
    descript = [cham_name,cham_lane]
    cham_type = []
    if tbody is not None:
        for tr in tbody.find_all(name="tr", limit=11):
            if isinstance(tr, bs4.element.Tag):
                tds = tr.find_all(name="td")
                # find champion type
                if tds[0] is not None:
                    kind = str(tds[0].find(name="a").string).replace("\n","")
                    kind = kind.rstrip().lstrip()
                    cham_type.append(kind)
                    cham_type.append(kind+"_Role Placement")
                # find average
                if tds[1] is not None:
                    average = str(tds[1].string).replace("\n","").replace(" ","")
                    descript.append(average)
                # find role placement
                if tds[2] is not None:
                    p_strong = str(tds[2].find(name="strong").string).replace("\n","").replace(" ","")
                    p_small = str(tds[2].find(name="small").string).replace("\n","").replace(" ","")
                    p=p_strong+p_small
                    descript.append(p)
        # overall placement
        if isinstance(tr, bs4.element.Tag):
            tr = tbody.find_all(name="tr")
            p_strong = str(tr[-1].find(name="strong").string).replace("\n","").replace(" ","")
            p_small = str(tr[-1].find(name="small").string).replace("\n","").replace(" ","")
            p=p_strong+p_small
            descript.append(p)
    return cham_type,descript
```
### Create dataframe
Use `get_championgg_dataframe()` to create dataframe
```python
def get_championgg_dataframe():
    urls = get_championgg_url()
    data_list = []
    # progress bar
    for url in tqdm(urls):
        cham_type , cham_list = get_championgg_data(url)
        data_list.append(cham_list)
    # Name dataframe columns
    df_columns_name = ["Champion Name","Lane"]+cham_type+["Overall Placement"]
    dataframe = pd.DataFrame(data_list)
    dataframe.columns = df_columns_name
    return dataframe
```
