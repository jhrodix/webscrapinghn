# Factor cambiario HN - BCH
## Historico Factor Cambiario


```python
from datetime import datetime
import requests as req
import xml.etree.ElementTree as ET 
import pandas as pd

BCH_URL="https://www.bch.hn/_api/web/lists/GetByTitle('LST-INDICADOR-TIPOCAMBIO')/items?$top=5000&$select=Fecha,Etiqueta1,Valor1&$orderby=FechaPublicacion1 desc"

def getWsFactorCambiario():
	""" Extrayendo datos """
	res=req.get(BCH_URL,params={'Accept':'application/json; odata=verbose'})
	tree = ET.fromstring(res.content)
	return [(tree[i][6][0][0].text, tree[i][6][0][1].text, tree[i][6][0][2].text) for i in range(3,len(tree))]

def getFactorCambiarioSinCambios():
	""" Generando el dataset """
	data=pd.DataFrame(getWsFactorCambiario())
	data=data.rename(columns={0:'Fecha',1:'Tipo',2:'Factor'})
	return data
  
def getFactorCambiario():
	""" Limpiando el dataset """
	data = getFactorCambiarioSinCambios()
	data['Fecha']=data['Fecha'].str.strip()
	data['Dia']=data['Fecha'].apply(lambda x: x.split(' ')[0])
	data['Fecha']=data['Fecha'].apply(lambda x: x.split(' ')[1])
	data['Fecha']=data['Fecha'].apply(lambda x: pd.to_datetime(x,format='%d/%m/%Y'))
	data['Factor']=data['Factor'].astype(float)
	data.loc[data['Tipo']=='Valor de Compra','Tipo']='Compra'
	fechas = pd.DataFrame({'Fecha':pd.date_range(start=min(data['Fecha']),end=max(data['Fecha']))})
	data=fechas.merge(data,on='Fecha',how='left')
	data=data.fillna(method='pad')
	data=data.sort_values(by='Fecha')
	return(data)

data=getFactorCambiario()
data.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Tipo</th>
      <th>Factor</th>
      <th>Dia</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-02-09</td>
      <td>Compra</td>
      <td>24.0733</td>
      <td>Martes</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-02-09</td>
      <td>Compra</td>
      <td>25.0733</td>
      <td>Martes</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-02-10</td>
      <td>Compra</td>
      <td>24.0718</td>
      <td>Miércoles</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-02-11</td>
      <td>Compra</td>
      <td>24.0684</td>
      <td>Jueves</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-02-12</td>
      <td>Compra</td>
      <td>25.0647</td>
      <td>Viernes</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2021-02-13</td>
      <td>Compra</td>
      <td>25.0647</td>
      <td>Viernes</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2021-02-14</td>
      <td>Compra</td>
      <td>25.0647</td>
      <td>Viernes</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2021-02-15</td>
      <td>Compra</td>
      <td>24.0778</td>
      <td>Lunes</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021-02-16</td>
      <td>Compra</td>
      <td>24.0708</td>
      <td>Martes</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2021-02-17</td>
      <td>Compra</td>
      <td>24.0670</td>
      <td>Miércoles</td>
    </tr>
  </tbody>
</table>
</div>




```python
import plotly.express as px

df = data
fig = px.line(df, x='Fecha', y="Factor",labels={'Factor':'Precio Compra (lps)','Fecha':'Fecha'})

fig.update_xaxes(
    rangeslider_visible=True,
    rangeselector=dict(
        buttons=list([
            dict(count=1, label="1m", step="month", stepmode="backward"),
            dict(count=6, label="6m", step="month", stepmode="backward"),
            dict(count=1, label="YTD", step="year", stepmode="todate"),
            dict(count=1, label="1y", step="year", stepmode="backward"),
            dict(step="all")
        ])
    )
)
fig.write_html('fcwidget.html')
fig.show()
```



# ver [widget](./fcwidget.html)
