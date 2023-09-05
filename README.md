#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys


def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'SCM.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
matches['season'] = matches['date'].str[:4].astype(int)
data = [go.Histogram(x=matches['season'], marker=dict(color='#EB89B5', line=dict(color='#000000', width=1)), opacity=0.75)]
layout = go.Layout(title='Matches In Every Season ',xaxis=dict(title='Season',tickmode='linear'),
                    yaxis=dict(title='Count'),bargap=0.2, plot_bgcolor='rgb(245,245,245)')

fig = go.Figure(data=data, layout=layout)
iplot(fig)
matches_played=pd.concat([matches['team1'],matches['team2']])
matches_played=matches_played.value_counts().reset_index()
matches_played.columns=['Team','Total Matches']
matches_played['wins']=matches['winner'].value_counts().reset_index()['winner']

matches_played.set_index('Team',inplace=True)
totm = matches_played.reset_index().head(8)

trace = go.Table(
    header=dict(values=["Team","Total Matches","Wins"],
                fill = dict(color='#ff96ea'),
                font = dict(color=['rgb(45, 45, 45)'] * 5, size=14),
                align = ['center'],
               height = 30),
    cells=dict(values=[totm['Team'], totm['Total Matches'], totm['wins']],
               fill = dict(color=['rgb(235, 193, 238)', 'rgba(228, 222, 249, 0.65)']),
               align = ['center'], font_size=13, height=25))

layout = dict(
    width=750,
    height=420,
    autosize=False,
    title='Total Matches vs Wins per team',
    margin = dict(t=100),
    showlegend=False,    
)

fig1 = dict(data=[trace], layout=layout)
iplot(fig1)
view rawIPL analysis.py hosted with ❤ by GitHub
trace1 = go.Bar(x=matches_played.index,y=matches_played['Total Matches'],
                name='Total Matches',opacity=0.4)

trace2 = go.Bar(x=matches_played.index,y=matches_played['wins'],
                name='Matches Won',marker=dict(color='red'),opacity=0.4)

trace3 = go.Bar(x=matches_played.index,
               y=(round(matches_played['wins']/matches_played['Total Matches'],3)*100),
               name='Win Percentage',opacity=0.6,marker=dict(color='gold'))

data = [trace1, trace2, trace3]

layout = go.Layout(title='Match Played, Wins And Win Percentage',xaxis=dict(title='Team'),
                   yaxis=dict(title='Count'),bargap=0.2,bargroupgap=0.1, plot_bgcolor='rgb(245,245,245)')

fig = go.Figure(data=data, layout=layout)
iplot(fig)
venue_matches=matches.groupby('venue').count()[['id']].sort_values(by='id',ascending=False).head()
ser = pd.Series(venue_matches['id']) 
venue_matches=matches.groupby('venue').count()[['id']].reset_index()

data = [{"y": venue_matches['id'],"x": venue_matches['venue'], 
          "marker": {"color": "lightblue", "size": 12},
         "line": {"color": "red","width" : 2,"dash" : 'dash'},
          "mode": "markers+lines", "name": "Women", "type": "scatter"}]

layout = {"title": "Stadiums Vs. Matches", 
          "xaxis": {"title": "Matches Played", }, 
          "yaxis": {"title": "Stadiums"},
          "autosize":False,"width":900,"height":700,"plot_bgcolor":"rgb(245,245,245)"}

fig = go.Figure(data=data, layout=layout)
iplot(fig)
data = [go.Bar(
    x = matches["toss_decision"].value_counts().index,
    y = matches["toss_decision"].value_counts().values,
    marker = dict(line=dict(color='#000000', width=1))
)]

layout = go.Layout(
   {
      "title":"Most Likely Decision After Winning Toss",
       "xaxis":dict(title='Decision'),
       "yaxis":dict(title='Number of Matches'),
       "plot_bgcolor":'rgb(245,245,245)'
   }
)
fig = go.Figure(data=data,layout = layout)
iplot(fig)
batsmen = matches[['id','season']].merge(deliveries, left_on = 'id', right_on = 'id', how = 'left').drop('id', axis = 1)
season=batsmen.groupby(['season'])['total_runs'].sum().reset_index()

avgruns_each_season=matches.groupby(['season']).count().id.reset_index()
avgruns_each_season.rename(columns={'id':'matches'},inplace=1)
avgruns_each_season['total_runs']=season['total_runs']
avgruns_each_season['average_runs_per_match']=avgruns_each_season['total_runs']/avgruns_each_season['matches']
Season_boundaries=batsmen.groupby("season")["batsman_runs"].agg(lambda x: (x==6).sum()).reset_index()
fours=batsmen.groupby("season")["batsman_runs"].agg(lambda x: (x==4).sum()).reset_index()
Season_boundaries=Season_boundaries.merge(fours,left_on='season',right_on='season',how='left')
Season_boundaries=Season_boundaries.rename(columns={'batsman_runs_x':'6"s','batsman_runs_y':'4"s'})

Season_boundaries['6"s'] = Season_boundaries['6"s']*6
Season_boundaries['4"s'] = Season_boundaries['4"s']*4
Season_boundaries['total_runs'] = season['total_runs']

trace1 = go.Bar(
    x=Season_boundaries['season'],
    y=Season_boundaries['total_runs']-(Season_boundaries['6"s']+Season_boundaries['4"s']),
    marker = dict(line=dict(color='#000000', width=1)),
    name='Remaining runs',opacity=0.6)

trace2 = go.Bar(
    x=Season_boundaries['season'],
    y=Season_boundaries['4"s'],
    marker = dict(line=dict(color='#000000', width=1)),
    name='Run by 4"s',opacity=0.7)

trace3 = go.Bar(
    x=Season_boundaries['season'],
    y=Season_boundaries['6"s'],
    marker = dict(line=dict(color='#000000', width=1)),
    name='Run by 6"s',opacity=0.7)


data = [trace1, trace2, trace3]
layout = go.Layout(title="Run Distribution per year",barmode='stack',xaxis = dict(tickmode='linear',title="Year"),
                                    yaxis = dict(title= "Run Distribution"), plot_bgcolor='rgb(245,245,245)')

fig = go.Figure(data=data, layout=layout)
iplot(fig)
high_scores=deliveries.groupby(['id', 'inning','batting_team','bowling_team'])['total_runs'].sum().reset_index() 
high_scores=high_scores[high_scores['total_runs']>=200]
hss = high_scores.nlargest(10,'total_runs')

trace = go.Table(
    header=dict(values=["Inning","Batting Team","Bowling Team", "Total Runs"],
                fill = dict(color = 'red'),
                font = dict(color = 'white', size = 14),
                align = ['center'],
               height = 30),
    cells=dict(values=[hss['inning'], hss['batting_team'], hss['bowling_team'], hss['total_runs']],
               fill = dict(color = ['lightsalmon', 'rgb(245, 245, 249)']),
               align = ['center'], font_size=13))

layout = dict(
    width=830,
    height=410,
    autosize=False,
    title='Highest scores of IPL',
    showlegend=False,    
)

fig1 = dict(data=[trace], layout=layout)
iplot(fig1)
