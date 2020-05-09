# keralacoronatracker
Backend program for a corona tracker website. Helps you to update graphs/data in the website via colab. So updation can be done anywhere. Mobile or tablet or PC. Avoids the need for a server that is able to run python program in it. Or the need of a computer that has to be on 24X7
1. Install Chart Studio (As you use colab, you have to install chart_studio every single time the remote server gets on)
ps. Chart-studio is the heart of this project. This brings you pretty graphs with minimum work
```python
pip install chart_studio
```
2.Import my google account (I am saving & retrieving some data from my drive too. This is to provide access to colab for the same)
```python
from google.colab import drive
drive.mount('/content/drive')
```
3. Importing all libraries
```python
import pandas as pd
import re
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import chart_studio
import pytz
tz = pytz.timezone('Asia/Calcutta')
from datetime import datetime as dt
```

4. Setting credentials for Chart-studio
```python
username = 'xxx'
api_key ='xxx'
chart_studio.tools.set_credentials_file(username, api_key)
import chart_studio.plotly as py
import chart_studio.tools as tls
```

5. Webcrawling from the central govt website for getting data
```python
df1 = pd.read_html('https://www.mohfw.gov.in', header=0, index_col=0)[0] #Reading the 1st table in the site
df1.drop(df1.tail(4).index,inplace=True) #removing last 4 rows in the site as its unwanted for me
df1 = df1.reset_index() 
df1.columns = ['Np', 'State',
       'Total',
       'Recovered', 'Death'] #Renaming the columns
#Renaming row names as I thought these state names are too long :P
df1 = df1.replace(to_replace ='Andaman and Nicobar Islands', 
                 value ="An & Nic Islands") 
df1 = df1.replace(to_replace ='Jammu and Kashmir', 
                 value ="Jammu & K") 
df1 = df1.replace(to_replace ='Himachal Pradesh', 
                 value ="Himachal P") 
df1 = df1.replace(to_replace ='Arunachal Pradesh', 
                 value ="Arunachal P") 
```

6. Calculating still infected count. (govt web provides active, recovered and deaths alone)
```python
df1['Total']=pd.to_numeric(df1['Total'])
df1['Recovered']=pd.to_numeric(df1['Recovered'])
df1['Death']=pd.to_numeric(df1['Death'])
df1['Infected'] = df1['Total'] - df1['Recovered'] - df1['Death'] 
```

7. Plotting my graph in plotly using extracted data
```python
import plotly.graph_objects as go
fig4 = go.Figure(go.Bar(x = df1['State'], y=df1['Infected'], name='Infected'))
fig4.add_trace(go.Bar(x = df1['State'], y=df1['Recovered'], name='Recovered',marker=go.bar.Marker(color='rgb(128, 255, 0)')))
fig4.add_trace(go.Bar(x = df1['State'], y=df1['Death'], name='Death', hoverinfo='text',marker=go.bar.Marker(color='rgb(255, 0, 0)'), text=[('Total: ' + str(a) + '<br>Recovered:' + str(b) + '<br>Infected:' + str(c) +'<br>Deaths:' + str(d)) for a, b, c, d in zip(df1['Total'], df1['Recovered'], df1['Infected'], df1['Death'])]))
fig4.update_traces(textposition='outside',textfont_size=20)
fig4.update_layout(barmode='stack', xaxis={'categoryorder':'total ascending'},  title_text = 'INDIA | Statewise Figures | Last updated: {}'.format(dt.now(tz)),
                              
annotations=[
        dict(
            x="Kerala", #I wanted to highlight kerala in the graph
            y=0,
            xref="x",
            yref="y",
            text="",
            showarrow=True,
            arrowhead=5,
            ax=0,
            ay=+20
        )
    ])
fig4.show()
```

8. Updating the same in cloud so that it gets updated in my mobile app and website
```python
py.plot(fig4, filename = 'deaths', auto_open = True)
```

See it in action at www.gocoronago.tech
The one I have discussed here is the 2nd last graph in homepage.
i.e., the India statewise statistics
