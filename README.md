# The-COVID-View-Cases-Deaths-Recoveries-in-Data
import pandas as pd
import plotly.express as px
from dash import Dash, dcc, html, Input, Output

# Sample Data (Replace this with your actual dataset or CSV file)
data = {
    "date": ["2020-03-01", "2020-03-02", "2020-03-03", "2020-03-04", "2020-03-05"],
    "country": ["USA", "USA", "USA", "USA", "USA"],
    "new_cases": [5, 10, 20, 30, 15],
    "deaths": [0, 1, 2, 3, 1],
    "recovered": [1, 2, 4, 6, 3]
}
df = pd.DataFrame(data)
df['date'] = pd.to_datetime(df['date'])

# Create Dash app
app = Dash(__name__)
app.title = "COVID-19 Dashboard"

# Layout
app.layout = html.Div([
    html.H1("🦠 COVID-19 Dashboard", style={'textAlign': 'center'}),
    
    dcc.Dropdown(
        id='country-dropdown',
        options=[{'label': c, 'value': c} for c in df['country'].unique()],
        value=df['country'].unique()[0],
        clearable=False
    ),

    html.Br(),

    dcc.Graph(id='line-chart'),

    dcc.Graph(id='bar-chart'),

    dcc.Graph(id='pie-chart'),
])

# Callbacks
@app.callback(
    [Output('line-chart', 'figure'),
     Output('bar-chart', 'figure'),
     Output('pie-chart', 'figure')],
    [Input('country-dropdown', 'value')]
)
def update_graphs(selected_country):
    filtered = df[df['country'] == selected_country]

    # Line Chart: Daily new cases
    line_fig = px.line(
        filtered,
        x='date',
        y='new_cases',
        title=f'Daily New COVID-19 Cases in {selected_country}',
        labels={'new_cases': 'New Cases'}
    )

    # Bar Chart: Total cases by country
    total_cases = df.groupby('country')['new_cases'].sum().reset_index()
    bar_fig = px.bar(
        total_cases.sort_values(by='new_cases', ascending=False),
        x='country',
        y='new_cases',
        title='Total COVID-19 Cases by Country',
        labels={'new_cases': 'Total Cases'}
    )

    # Pie Chart: Recovery vs Death Rate (Total values for the selected country)
    totals = filtered[['deaths', 'recovered']].sum()
    pie_fig = px.pie(
        names=['Deaths', 'Recovered'],
        values=[totals['deaths'], totals['recovered']],
        title=f'Recovery vs Death Rate in {selected_country}'
    )

    return line_fig, bar_fig, pie_fig

# Run the app
if __name__ == '__main__':
    app.run_server(debug=True)
