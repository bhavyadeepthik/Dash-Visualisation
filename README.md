# Dash-Visualisation
Dash /Plotly Assignment
#Authour@ Bhavya-Deepthi
# import needed libraries


import pandas as pd

import numpy as np

import dash

import dash_core_components as dcc

import dash_html_components as html

import plotly.graph_objs as go



# Load the dataframe with flight dataset

df = pd.read_csv("flights.csv",low_memory=False)

df.head()



#Load the Dataframe with airlines data

df_airline = pd.read_csv("airlines.csv",low_memory=False)

df_airline.head()



#Merge Airline Information with Flight dataset to know about the name of the operating airlines

df = pd.merge(df,df_airline, left_on='AIRLINE', right_on = 'IATA_CODE')

df.insert(loc=5, column='AIRLINE', value=df.AIRLINE_y)

df = df.drop(['AIRLINE_y','IATA_CODE'], axis=1)

df.head()



# Merge the Airport Data with Flight Dataset to know about the airport information

df_airport = pd.read_csv('airports.csv')

df = pd.merge(df,df_airport[['IATA_CODE','AIRPORT','CITY']], left_on='ORIGIN_AIRPORT', right_on = 'IATA_CODE')

df = df.drop(['IATA_CODE'], axis=1)

df = pd.merge(df,df_airport[['IATA_CODE','AIRPORT','CITY']], left_on='DESTINATION_AIRPORT', right_on = 'IATA_CODE')

df = df.drop(['IATA_CODE'], axis=1)

df.head()



app = dash.Dash()



# To get the airlines travelled distance

dff = df.groupby('AIRLINE').sum().DISTANCE.sort_values(ascending=False)

trace1 = go.Bar(x=dff.index,y=dff.values,marker=dict(color = dff.values,colorscale='Jet',showscale=True))



# Get the Average cancellation of airlines

dff = df.groupby('AIRLINE')[['CANCELLED']].mean().sort_values(by='CANCELLED', 

                                                    ascending=False).round(3)



trace2 = go.Scatter(x=dff.index,y=dff.CANCELLED,mode='markers',marker=dict(symbol = 'square',

        sizemode = 'diameter',sizeref = 1,size = 30,color = dff.CANCELLED,colorscale='blues',showscale=True

    )

)



# Get the Average Arrival delay minutes

dff=df.groupby('AIRLINE').ARRIVAL_DELAY.mean().sort_values(ascending=False)

label = dff.index

size = dff.values

colors = ['skyblue', '#FEBFB3', '#96D38C', '#D0F9B1', 'gold', 'orange', 'lightgrey', 

          'lightblue','lightgreen','aqua']

trace3= go.Pie(labels=label, values=size, marker=dict(colors=colors),hole = .2)



# To create the HeatMap

flight_volume = df.pivot_table(index="CITY_x",columns="DAY_OF_WEEK",

                               values="DAY",aggfunc=lambda x:x.count())

fv = flight_volume.sort_values(by=1, ascending=False)[:8]

fv.index = np.where(fv.index=='Dallas-Fort Worth','Dallas', fv.index)



trace4 = go.Heatmap(z=[fv.values[1],fv.values[2],fv.values[3],fv.values[4],

                      fv.values[5],fv.values[6],fv.values[7]],

                   x=['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday',

                      'Saturday','Sunday'],

                   y=fv.index.values, colorscale='Reds'

                  )



# Get the Average Departure delay minutes

dff=df.groupby('AIRLINE').DEPARTURE_DELAY.mean().sort_values(ascending=False)

label = dff.index

size = dff.values



app.layout = html.Div([

    

    dcc.Graph(

        id='bar chart',

        figure = {

            'data': [trace1],

            'layout': go.Layout(xaxis=dict(tickangle=15),

                    title='Airlines travelled distance in 2015', 

                   yaxis = dict(title = '# In Kilometers ')

            )

        }

    ),

 dcc.Graph(

        id='Scatter Plot',

        figure = {

            'data': [trace2],

            'layout': go.Layout(xaxis=dict(tickangle=15),

                    title='Rate of cancellation', 

                   yaxis = dict(title = 'Airline Cancellation')

            )

        }

    ),

    dcc.Graph(

        id='Pie chart',

        figure = {

            'data': [trace3],

            'layout': go.Layout(xaxis=dict(tickangle=15),

                    title='Delays by Airlines'

            )

        }

    ),

    dcc.Graph(

        id='Heatmap',

        figure = {

            'data': [trace4],

            'layout': go.Layout(xaxis=dict(tickangle=15),

                    title='Arrival delays'

            )

        }

    ),

    dcc.Graph(figure={'data': [

            {

                'x': label,

                'y': size,

                'type': 'scatter',

                'mode': 'markers',

                'error_y': {

                    'type': 'data',

                    'symmetric': False,

                    'arrayminus': size,

                    'array': [0] * len(size), 

                    'width': 0

                },

                'marker': {

                    'size': 8

                }

            },

        ],'layout': go.Layout(xaxis=dict(tickangle=15),

                    title='Average Departure delay minutes in 2015', 

                   yaxis = dict(title = '# In Minutes '))

})

])



if __name__ == '__main__':

    app.run_server(port=4051)

