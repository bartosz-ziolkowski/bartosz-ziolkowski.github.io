---
layout: post
title: "Final Project - Explainer"
classes: wide
date: 2024-05-06 12:00:10 +0100
categories: jekyll update
---

# Authors - Group 48

- Bartosz Ziolkowski, s230080
- Kristoffer Plehn, s203777

## 1. Motivation

We used two datasets from [Statistikbanken.dk: ](https://www.statistikbanken.dk/)
- Immigration by sex, age, country of origin and citizenship (1980-2023)      
- Emigration by sex, age, country of destination and citizenship (1980-2023)

We chose them because they contain data on the number and some metrics of migrants in Denmark, which is what we wanted to analyze. In addition, they come from an authorized source.

Our goal in analyzing the datasets was to acquaint the end user with immigration to Denmark and emigration from Denmark between 2015 and 2023, considering individuals aged 0-80 and EU/EEA countries.

## 2. Basic Stats

The dataset on immigration has a total size of 494 KB and 10.045 rows. In turn, the dataset on emigration has a total size of 491 KB and 10.045 rows as well. 

Both datasets have 13 columns: `OriginCountry`/`Destination`, `Citizenship`, `Sex`, `Age`, `2015`, `2016`, `2017`, `2018`, `2019`, `2020`, `2021`, `2022`, `2023`. 

The user interface of the website with the datasets allows the user to download the data by selecting desired columns and other grouping values e.g. specific countries, years, citizenships, age, and sex. We were interested in the period from 2015-2023, EU/EEA countries and migrants aged 0-80.

## 3. Data Analysis

### 3.1 Code on the Initial Overview on Migration

```python
import pandas as pd
import plotly.graph_objects as go

immigration_data = pd.read_csv('Immigration_2015-2023.csv', sep=';')
emigration_data = pd.read_csv('Emigration_2015-2023.csv', sep=';')

# Extract years and data
years = immigration_data.columns[4:]
immigration_total = immigration_data[years].sum()
emigration_total = emigration_data[years].sum()
migration_balance = immigration_total - emigration_total

fig = go.Figure()

fig.add_trace(go.Scatter(x=years, y=immigration_total, mode='lines+markers', name='Immigration', line=dict(color='blue')))
fig.add_trace(go.Scatter(x=years, y=emigration_total, mode='lines+markers', name='Emigration', line=dict(color='red')))
fig.add_trace(go.Scatter(x=years, y=migration_balance, mode='lines+markers', name='Balance', line=dict(color='green')))

fig.update_layout(
    title='Migration in Denmark Over the Years',
    xaxis=dict(title='Year', showgrid=True),
    yaxis=dict(title='Number of People', showgrid=True),
    legend=dict(x=0, y=1.0, bgcolor='rgba(255, 255, 255, 0)', bordercolor='rgba(255, 255, 255, 0)'),
    plot_bgcolor='rgba(0, 0, 0, 0)',
)

fig.show()

fig.to_html(full_html=False, include_plotlyjs='cdn')
fig.write_html("output.html", full_html=False, include_plotlyjs='cdn')
```

### 3.2 Code on the Share of Top 15 Countries in Migration

```python
import pandas as pd
import matplotlib.pyplot as plt

immigration_data = pd.read_csv('Immigration_2015-2023.csv', sep=';')
emigration_data = pd.read_csv('Emigration_2015-2023.csv', sep=';')

# Select only numeric columns for both immigration and emigration data
numeric_columns_immigration = immigration_data.select_dtypes(include='number')
numeric_columns_emigration = emigration_data.select_dtypes(include='number')

# Calculate total immigration and emigration for each country
total_immigration_by_country = numeric_columns_immigration.groupby(immigration_data['OriginCountry']).sum().sum(axis=1)
total_emigration_by_country = numeric_columns_emigration.groupby(emigration_data['Destination']).sum().sum(axis=1)

# Select the top 15 countries with the highest immigration and emigration numbers
top_15_countries_immigration = total_immigration_by_country.nlargest(15)
top_15_countries_emigration = total_emigration_by_country.nlargest(15)

# Define colors for charts
colors = [
    '#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd',
    '#8c564b', '#e377c2', '#7f7f7f', '#bcbd22', '#17becf',
    '#1a55FF', '#2aFF3A', '#FF5733', '#33FFF6', '#8E44AD'
]

# Calculate migration ratio
migration_ratio = top_15_countries_immigration - top_15_countries_emigration

# Sort migration ratio data in descending order
migration_ratio_sorted = migration_ratio.sort_values(ascending=False)

fig, (ax1, ax2, ax3) = plt.subplots(1, 3, figsize=(20, 8))

ax1.pie(top_15_countries_immigration, autopct='%1.1f%%', colors=colors, textprops={'color': 'white', 'weight': 'bold'})
ax1.set_title('Top 15 Countries by Total Immigration to Denmark', loc='center', fontsize=18)

ax2.pie(top_15_countries_emigration, autopct='%1.1f%%', colors=colors, textprops={'color': 'white', 'weight': 'bold'})
ax2.set_title('Top 15 Countries by Total Emigration from Denmark', loc='center', fontsize=18)

ax1.legend(top_15_countries_immigration.index, loc='upper left', bbox_to_anchor=(-0.1, 1), ncol=1)
ax2.legend(top_15_countries_emigration.index, loc='upper left', bbox_to_anchor=(-0.1, 1), ncol=1)

ax3.bar(migration_ratio_sorted.index, migration_ratio_sorted, color=colors)
ax3.set_title('Migration Ratio (Immigration - Emigration)', fontsize=18)
ax3.set_xlabel('Country', fontsize=14)
ax3.set_ylabel('Number of People', fontsize=14)
ax3.tick_params(axis='x', rotation=45)
ax3.grid(axis='y', linestyle='--', alpha=0.7)

plt.tight_layout()
plt.show()
```

### 3.3 Code on the Bokeh Plot on Migration

```python
import pandas as pd
from bokeh.plotting import figure, show, output_file
from bokeh.models import ColumnDataSource, CustomJS, RadioButtonGroup, FixedTicker, Legend, LegendItem
from bokeh.layouts import column
from bokeh.palettes import Spectral11

# Additional colors for better visualization
Spectral15 = tuple(Spectral11) + ('#6a51a3', '#807dba', '#9e9ac8', '#000000')

emigration_data = pd.read_csv("Emigration_2015-2023.csv", delimiter=';', thousands=',')
immigration_data = pd.read_csv("Immigration_2015-2023.csv", delimiter=';', thousands=',')

# Reshape data for easy plotting
emigration_data = emigration_data.melt(id_vars=['Destination', 'Citizenship', 'Sex', 'Age'], 
                                         var_name='Year', value_name='Emigrants')
immigration_data = immigration_data.melt(id_vars=['OriginCountry', 'Citizenship', 'Sex', 'Age'], 
                                         var_name='Year', value_name='Immigrants')

# Convert 'Year' column to integer type
emigration_data['Year'] = emigration_data['Year'].astype(int)
immigration_data['Year'] = immigration_data['Year'].astype(int)

total_immigration_by_country = immigration_data.groupby(['OriginCountry']).sum()
total_emigration_by_country = emigration_data.groupby(['Destination']).sum()

top_15_countries_immigration = total_immigration_by_country['Immigrants'].nlargest(15).index
top_15_countries_emigration = total_emigration_by_country['Emigrants'].nlargest(15).index

# Filter data for top 15 countries
immigration_data = immigration_data[immigration_data['OriginCountry'].isin(top_15_countries_immigration)]
emigration_data = emigration_data[emigration_data['Destination'].isin(top_15_countries_emigration)]

# Pivot data for easy plotting
immigration_data = immigration_data.pivot_table(index='Year', columns='OriginCountry', values='Immigrants', aggfunc='sum').fillna(0)
emigration_data = emigration_data.pivot_table(index='Year', columns='Destination', values='Emigrants', aggfunc='sum').fillna(0)

# Create ColumnDataSource for both immigration and emigration data
imm_src = ColumnDataSource(immigration_data)
emg_src = ColumnDataSource(emigration_data)

# Create figures for plotting
imm_chart = figure(width=1200, x_axis_label="Year", y_axis_label="Number of Immigrants",
                   title="Immigration to Denmark by Country (2015-2023)", x_range=(2015, 2023))
emg_chart = figure(width=1200, x_axis_label="Year", y_axis_label="Number of Emigrants",
                   title="Emigration from Denmark by Country (2015-2023)", x_range=(2015, 2023), visible=False)

imm_legend = Legend(items=[])
emg_legend = Legend(items=[])

# Plot lines for each country in immigration data
for i, country in enumerate(top_15_countries_immigration):
    line = imm_chart.line(x='Year', y=country, source=imm_src, line_width=2,
                          color=Spectral15[i])
    imm_legend.items.append(LegendItem(label=str(country), renderers=[line]))

# Plot lines for each country in emigration data
for i, country in enumerate(top_15_countries_emigration):
    line = emg_chart.line(x='Year', y=country, source=emg_src, line_width=2,
                          color=Spectral15[i])
    emg_legend.items.append(LegendItem(label=str(country), renderers=[line]))


imm_legend.click_policy = "hide"  
imm_legend.location = "top_left"
emg_legend.click_policy = "hide"
emg_legend.location = "top_right"


imm_chart.add_layout(imm_legend, 'right')
emg_chart.add_layout(emg_legend, 'right')

# Set fixed tickers for x-axis
years = list(range(2015, 2024)) 
imm_chart.xaxis.ticker = FixedTicker(ticks=years)
emg_chart.xaxis.ticker = FixedTicker(ticks=years)

# Define callback function for radio button group
callback = CustomJS(args=dict(imm_chart=imm_chart, emg_chart=emg_chart), code="""
    if (cb_obj.active == 0) {
        emg_chart.visible = false;
        imm_chart.visible = true;
    } else {
        imm_chart.visible = false;
        emg_chart.visible = true;
    }
""")

# Create radio button group for selecting immigration or emigration chart
radio_button_group = RadioButtonGroup(labels=["Immigration", "Emigration"], active=0)
radio_button_group.js_on_change('active', callback)

# Arrange plots and radio button group in a column layout
layout = column(radio_button_group, imm_chart, emg_chart)

output_file("ImmigrationEmigrationByCountry.html")
show(layout)
```

### 3.4 Code on Gender Trends in Migration

```python
import pandas as pd
import plotly.graph_objects as go

i_data = pd.read_csv("Immigration_2015-2023.csv", sep=';') 
e_data = pd.read_csv("Emigration_2015-2023.csv", sep=';')

# Remove 'year' or 'years' from age column
i_data['Age'] = i_data['Age'].str.replace(r'\s*years?', '', regex=True).astype(int)
e_data['Age'] = e_data['Age'].str.replace(r'\s*years?', '', regex=True).astype(int)

x = ['2015', '2016', '2017', '2018', '2019', '2020', '2021', '2022', '2023']

# Group data by gender and sum counts for each year
i_data_count = i_data.groupby('Sex')[x].sum().T
e_data_count = e_data.groupby('Sex')[x].sum().T

# Create immigration bar traces
immigration_trace = go.Bar(x=i_data_count.index, y=i_data_count['Men'], name='Men', marker_color='steelblue', visible=True)
immigration_trace_women = go.Bar(x=i_data_count.index, y=i_data_count['Women'], name='Women', marker_color='hotpink', visible=True)

# Create emigration bar traces
emigration_trace = go.Bar(x=e_data_count.index, y=e_data_count['Men'], name='Men', marker_color='steelblue', visible=False)
emigration_trace_women = go.Bar(x=e_data_count.index, y=e_data_count['Women'], name='Women', marker_color='hotpink', visible=False)

# Create dropdown menu
updatemenus = [{'buttons': [
    {'method': 'update',
     'label': 'Immigration',
     'args': [{'visible': [True, True, False, False]}, {'title': 'Immigration by Gender Over the Years'}]},
    {'method': 'update',
     'label': 'Emigration',
     'args': [{'visible': [False, False, True, True]}, {'title': 'Emigration by Gender Over the Years'}]}
],
'direction': 'down',
'pad': {'r': 10, 't': 10},
'x': 0.8,
'xanchor': 'left',
'y': 1.2,
}]

layout = {'updatemenus': updatemenus,
          'title': 'Immigration by Gender Over the Years',
          'xaxis': {'title': 'Year'},
          'yaxis': {'title': 'Number of People'}}

fig = go.Figure(data=[immigration_trace, immigration_trace_women, emigration_trace, emigration_trace_women], layout=layout)

fig.show()

fig.write_html("ImmigrationEmigrationByGender.html", full_html=False, include_plotlyjs='cdn')
```

### 3.5 Code on Age Trends in Migration

```python
import matplotlib.pyplot as plt
import pandas as pd

# Define intervals for age groups
interval_size = 10
max_age = 80
age_bins = range(0, max_age + 1, interval_size)
age_labels = [f"{i}-{i+interval_size-1}" for i in range(0, max_age, interval_size)]

# Create empty DataFrames for aggregated immigration and emigration data
aggregated_i_data = pd.DataFrame(index=age_labels, columns=range(2015, 2024))
aggregated_e_data = pd.DataFrame(index=age_labels, columns=range(2015, 2024))

# Group individual data by age and sum the number of people for each year
grouped_i_data = i_data.groupby(pd.cut(i_data['Age'], bins=age_bins, labels=age_labels, right=False), observed=False).sum()
grouped_e_data = e_data.groupby(pd.cut(e_data['Age'], bins=age_bins, labels=age_labels, right=False), observed=False).sum()

# Sum the number of people in each age group for each year
for age_group in age_labels:
    for year in range(2015, 2024):
        aggregated_i_data.loc[age_group, year] = grouped_i_data.loc[age_group, str(year)]
        aggregated_e_data.loc[age_group, year] = grouped_e_data.loc[age_group, str(year)]

# Calculate the total number of immigrants and emigrants for each age group over all years
total_i_data = aggregated_i_data.sum(axis=1)
total_e_data = aggregated_e_data.sum(axis=1)

# Create a figure and axis for plotting
fig, ax = plt.subplots(figsize=(10, 6))

# Plot total immigration
total_i_data.plot(kind='bar', color='royalblue', position=0, width=0.4, label='Immigrants', ax=ax)

# Plot total emigration
total_e_data.plot(kind='bar', color='darkturquoise', position=1, width=0.4, label='Emigrants', ax=ax)

ax.set_xlabel("Age Group")
ax.set_ylabel("Number of People")
ax.set_title("Total Immigration and Emigration by Age Group (2015-2023)", fontsize=12)
ax.grid(axis='y', linestyle='-', color='lightgrey')

ax.legend(loc='upper left', bbox_to_anchor=(1, 1))

plt.xticks(rotation=0)
plt.tight_layout()

plt.show()
```

### 3.6 Code on Top 5 Countries for Age Groups

```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import plotly.express as px

immigration_data = pd.read_csv('Immigration_2015-2023.csv', sep=';')
emigration_data = pd.read_csv('Emigration_2015-2023.csv', sep=';')

# Remove 'year' or 'years' from age column
immigration_data['Age'] = immigration_data['Age'].str.replace(r'\D', '', regex=True).astype(int)
emigration_data['Age'] = emigration_data['Age'].str.replace(r'\D', '', regex=True).astype(int)

# Intervals for age groups
interval_size = 10
max_age = 80
age_bins = range(0, max_age + 1, interval_size)
age_labels = [f"{i}-{i+interval_size-1}" for i in range(0, max_age, interval_size)]

# Function to get top 5 countries for each age group
def get_top_countries(grouped_data):
    """
    Get top 5 countries for each age group based on immigration or emigration counts.

    Parameters:
    grouped_data (DataFrame): Grouped data by age and country.

    Returns:
    dict: Dictionary containing top 5 countries and their counts for each age group.
    """
    top_countries_data = {}
    country_column = grouped_data.index.names[-1]
    for age_group in age_labels:
        top_countries = grouped_data.loc[age_group].nlargest(5, ['2015', '2016', '2017', '2018', '2019', '2020', '2021', '2022', '2023'])
        sorted_top_countries = top_countries.sort_values(by='Total', ascending=False)
        top_countries_data[age_group] = {
            'countries': sorted_top_countries.index.get_level_values(country_column).tolist(),
            'values': sorted_top_countries['Total'].tolist()
        }
    return top_countries_data

# Group data by age and OriginCountry/Destination and sum the counts for each year
i_grouped_by_age_and_country = immigration_data.groupby([pd.cut(immigration_data['Age'], bins=age_bins, labels=age_labels, right=False), 'OriginCountry'], observed=False).sum()
e_grouped_by_age_and_country = emigration_data.groupby([pd.cut(emigration_data['Age'], bins=age_bins, labels=age_labels, right=False), 'Destination'], observed=False).sum()

# Add a 'Total' column for sorting purposes
i_grouped_by_age_and_country['Total'] = i_grouped_by_age_and_country.select_dtypes(include=[np.number]).sum(axis=1)
e_grouped_by_age_and_country['Total'] = e_grouped_by_age_and_country.select_dtypes(include=[np.number]).sum(axis=1)

# Get top 5 countries for immigration and emigration
i_top_countries_data = get_top_countries(i_grouped_by_age_and_country)
e_top_countries_data = get_top_countries(e_grouped_by_age_and_country)

bar_width = 0.1
space_between_groups = 0.3

# Calculate the x positions for each bar within its age group
x_positions = {}
for i, age_group in enumerate(age_labels):
    base_position = i * (5 * bar_width + space_between_groups)
    x_positions[age_group] = [base_position + j * bar_width for j in range(5)]

# Create traces for immigration and emigration
immigration_traces = []
emigration_traces = []

for age_group in age_labels:
    i_countries = i_top_countries_data[age_group]['countries']
    e_countries = e_top_countries_data[age_group]['countries']
    i_values = i_top_countries_data[age_group]['values']
    e_values = e_top_countries_data[age_group]['values']
    
    immigration_traces.append(go.Bar(
        x=x_positions[age_group],
        y=i_values,
        text=i_countries,
        name=f'Age:<br>{age_group}',
        visible=True,
        hovertemplate = "Count: %{y}",
        marker_color=px.colors.qualitative.Plotly[:len(i_values)],
        width=bar_width 
    ))
    
    emigration_traces.append(go.Bar(
        x=x_positions[age_group],
        y=e_values,
        visible=False,
        text=e_countries,
        hovertemplate = "Count: %{y}",
        name=f'Age:<br>{age_group}',
        marker_color=px.colors.qualitative.Plotly[len(i_values):len(i_values) + len(e_values)],
        width=bar_width 
    ))

# Create dropdown options
dropdown_options = [
    {
        'label': 'Immigration',
        'method': 'update',
        'args': [
            {'visible': [True] * len(immigration_traces) + [False] * len(emigration_traces)},
            {'title': 'Top 5 Countries by Age Group - Immigration'}
        ]
    },
    {
        'label': 'Emigration',
        'method': 'update',
        'args': [
            {'visible': [False] * len(immigration_traces) + [True] * len(emigration_traces)},
            {'title': 'Top 5 Countries by Age Group - Emigration'}
        ]
    }
]

# Update the layout with the dropdown menu
layout = go.Layout(
    title='Top 5 Countries by Age Group - Immigration',
    xaxis=dict(
        title='Age Group',
        tickvals=[i * (5 * bar_width + space_between_groups) + (2.5 * bar_width) for i in range(len(age_labels))],
        ticktext=age_labels,
        range=[-0.25, len(age_labels) * (5 * bar_width + space_between_groups)],
    ),
    yaxis=dict(title='Number of People'),
    barmode='group',
    showlegend=False,
    updatemenus=[{
        'buttons': dropdown_options,
        'direction': 'down',
        'pad': {'r': 10, 't': 10},
        'showactive': True,
        'x': 0.8,
        'xanchor': 'left',
        'y': 1.2,
        'yanchor': 'top'
    }],
    
)

fig = go.Figure(data=immigration_traces + emigration_traces, layout=layout)

fig.show()

fig.write_html("TopCountriesMigrationByAgeGroup.html", full_html=False, include_plotlyjs='cdn')
```

## 4. Genre

We chose to write the data story in Magazine Style. 

### 4.1 Visual Narrative

#### 4.1.1 Visual Structuring
* Consistent Visual Platform: The consistent visual platform is maintained throughout the visualization, with a clear layout, axis labels, title, and legend for interpretation to keep a similar structure throughout the post.

#### 4.1.2 Highlighting
* Feature Distinction: Features are distinguished using different colors for each country's data in the chart, and allowing readers to switch between charts.
* Zooming: Features on the interactive level can be zoomed in, in order to get further insights from the visualizations. 

### 4.2 Narrative Structure 

#### 4.2.1 Ordering
* Random Access: The order of the post is to be randomly explored and the viewer is therefore free to determine how they want to access the visualizations. 
  
#### 4.2.2 Interactivity
* Hover Highlighting / Details: Hover highlighting or details are implemented in charts to give better viewer experience. 
* Filtering / Selection / Search: Filtering and selection are provided through the buttons and mute policy, allowing users to switch between immigration and emigration data and compare different countries as well. 
* Very Limited Interactivity: The interactivity is limited to hovering and switching between two views of data (immigration and emigration) using buttons. 
* Tacit Tutorial: The buttons serve as a tacit tutorial, intuitively show users how to interact with the visualizations.
* Stimulating Default Views: The default view of the visualization is stimulating as it presents immigration data, where users can switch to emigration data for comparison. Furthermore, that charts with hover highlighting also prompting them to delve deeper into the data.

#### 4.2.3 Messaging
* Captions / Headlines: Captions or headlines are included to provide an overview of the visualizations.
* Summary / Synthesis: While the visualization provides a summary of immigration and emigration data, we explicitly synthesize the findings and compare them to the outside world. 

## 5. Visualizations

### 5.1 Visualization on the Initial Overview on Migration

This visualization consists of three line subplots. The first two aim to introduce and provide the reader with an overview of immigration and emigration trends over the period of 2015-2023. Meanwhile, the migration balance plot illustrates the overall migration situation in Denmark. In my opinion, line plots were a suitable choice for facilitating understanding by the average person because, in this case, the numbers of individuals over specific years clearly visualize the data history.

### 5.2 Visualization on the Share of Top 15 Countries in Migration

Pie charts are a suitable choice for representing the proportion of several elements in a given aspect, in this case, the share of the top 15 countries in both immigration and emigration - that's why we opted for them in this paragraph. Additionally, the bar chart was also a natural choice when it comes to illustrating the relationship of each country to its migration balance relative to Denmark. The graphs selected by us clearly and easily present the data described in this paragraph.

### 5.3 Visualization on the Bokeh Plot on Migration

To represent the trends of immigration and emigration in 15 countries, we chose the Bokeh plot because it adds the possibility of user interaction with the section. Additionally, the linear form of the plot is, in our opinion, the best choice for visualizing changes in a given value over time.

### 5.4 Visualization on Gender Trends in Migration

We have chosen a grouped barchart to visualize compare immigration and emigration by gender. This gives a static and great comparison option for the viewer, who quickly can read the answers. Furthermore, there is included a hovertool to further help the viewer to read the answers. These answers could easily be compared to the outside world to look for migration trends by gender. Instead of have two plots, we decided to have one interactive, where the viewer could choose between which migration they wanted to explore.

### 5.5 Visualization on Age Trends in Migration

For this visualization we wanted to discover which age groups were the primary ones and compare immigration and emigration directly in a grouped barchart. This gave a broad overview over the distribution of age and who contributed the most to the migration trends. It was interesting to see how much influence they had, which we then could delve further into in the next plot.

### 5.6 Visualization on Top 5 Countries for Age Groups

To delve further into the age groups, we did another grouped barchart with countries, where we found it interesting to compare the top 5 origin countries of immigrants and top 5 destinations of emigrants from Denmark. This visualization was also interactive, where the viewer can choose between migration for personal interests. The impact of the different age groups unveiled, which countries that contributed the most of who was getting in and out of Denmark. Thereby, we could look for patterns and understand why they arrived or left Denmark.

### 5.7 Why These Visualizations?

Selected charts regarding the initial review of immigration and emigration, as well as charts considering the main countries involved in migration in relation to Denmark, are presented in a straightforward and engaging manner, capturing readers' attention and illustrating the state of migration in the selected period.

The visualizations chosen for comparing gender and age groups in immigration and emigration serve the story well due to their ability to effectively convey comparisons and trends in the data. They effectively compare the distribution by showing relative contribution over time, which is crucial for understanding migration patterns. By highlighting differences and allowing for straighforward comparisons provides detailed insights into migration trends.

## 6. Discussion

Our visualization project aimed to explore migration patterns and trends to uncover the reality in Denmark. The chosen visualizations have proven to be effective in conveying the intended message and revealing insightful patterns in the data. However, to fully stimulate the viewer, the plots could have been more interactive showcasing each datapoint to be explored. We still find the level of complexity to be suitable for the story to be told and successfully enough to uncover the patterns and trends in the dataset. Even though, we have looked at the outside world to understand the overall trends in migration patterns, this could have further been explored to look for even more answers, which potentially could have uncovered additional insights. Furthermore, the findings of two datasets could lead to explore other datasets, explicitly looking at some of the most contributing countries' economic indicators to further understand the migration trends. 

Additionally, the visualization techniques effectively conveyed the story, but there may be opportunities to enhance the interactiveness and add more interactive elements to introduce even more playful features to the viewer. It could have been even more messaging or visual structuring tools to allow for more dynamically insights. This enables viewers to engage with the data more actively and customize their viewing experience according to their preferences. Furthermore, effectively guiding viewers through the narrative can ensure that key insights are communicated clearly. 

## 7. Contributions

### 7.1 Explainer Notebook

|   | s203777 | s230080 |
|---|--------|---------|
| Motivation | Reviewer | Main Contributor |
| Basic Stats | Reviewer | Main Contributor |
| Data Analysis | Reviewer | Main Contributor |
| Genre | Main Contributor | Reviewer |
| Visualizations | Main Contributor | Reviewer |
| Discussion | Main Contributor | Reviewer |

### 7.2 Blog Post

|   | s203777 | s230080 |
|---|--------|---------|
| Introduction | Reviewer | Main Contributor |
| Initial Overview on Migration | Reviewer | Main Contributor |
| Share of Top 15 Countries in Migration | Reviewer | Main Contributor |
| Track Immigration and Emigration for the Top 15 Countries | Reviewer | Main Contributor |
| Gender Trends in Immigration and Emigration | Main Contributor | Reviewer |
| Age Group Patterns in Immigration and Emigration | Main Contributor | Reviewer |
| Age Groups: Top 5 Immigrant Origins & Emigrant Destinations | Main Contributor | Reviewer |
| Conclusion | Main Contributor | Reviewer |

- [Click here to open the blog post on Assignment 2](http://localhost:4000/jekyll/update/2024/04/01/vehicle-theft-sanfrancisco.html)