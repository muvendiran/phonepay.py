# phonepay.py
import pandas as pd
import json
import os
import plotly.express as px
import requests
import subprocess
import plotly.graph_objects as go
from IPython.display import display
import psycopg2
import streamlit as st

st.set_page_config(layout='wide')

bala = psycopg2.connect(host = 'localhost',user = 'postgres', password='Abarna@21',port = 5432,database ='guvi')
cursor=bala.cursor()


cursor.execute("select * from Aggregate_transaction;")
bala.commit()
t1=cursor.fetchall()
Agg_Trans=pd.DataFrame(t1, columns=['State', 'Year', 'Quarter', 'Transaction_type', 'Transaction_count', 'Transaction_amount'])


cursor.execute("select * from Aggregate_user;")
bala.commit()
t2=cursor.fetchall()
Agg_user=pd.DataFrame(t2, columns=['State', 'Year', 'Quarter', 'Brands', 'Transaction_Count', 'Percentage'])


cursor.execute("select * from map_transaction;")
bala.commit()
t3=cursor.fetchall()
map_trans=pd.DataFrame(t3, columns=['State', 'Year', 'Quarter', 'District', 'Transaction_count', 'Transaction_amount'])


cursor.execute("select * from map_user;")
bala.commit()
t4=cursor.fetchall()
map_user=pd.DataFrame(t4, columns=['State', 'Year', 'Quarter', 'District', 'RegisteredUsers', 'AppOpens'])


cursor.execute("select * from top_trans;")
bala.commit()
t5=cursor.fetchall()
top_trans=pd.DataFrame(t5, columns=['State', 'Year', 'Quarter', 'Pincode', 'Transaction_Count', 'Transaction_Amount'])


cursor.execute("select * from top_user;")
bala.commit()
t6=cursor.fetchall()
top_user=pd.DataFrame(t6, columns=['State', 'Year', 'Quarter', 'Pincode', 'RegisteredUsers'])

def animate_all():
    
    url = "https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
    response = requests.get(url)
    data1 = json.loads(response.content)
    state_names_tra = [feature['properties']['ST_NM'] for feature in data1['features']]
    state_names_tra.sort()

    df_state_names_tra = pd.DataFrame({'State': state_names_tra})

    frames = []

    for year in Agg_Trans['Year'].unique():
        for quarter in Agg_Trans['Quarter'].unique():
            at1 = Agg_Trans[(Agg_Trans['Year'] == year) & (Agg_Trans['Quarter'] == quarter)]
            atf1 = at1[['State', 'Transaction_count']]
            atf1 = atf1.sort_values(by='State')
            atf1['Year'] = year
            atf1['Quarter'] = quarter
            frames.append(atf1)

    merged_df = pd.concat(frames)

    fig_tra = px.choropleth(
        merged_df, 
        geojson=data1, 
        locations='State', 
        featureidkey='properties.ST_NM', 
        color='Transaction_count',
        color_continuous_scale='Sunsetdark',
        range_color=(0,500000000),
        hover_name='State',
        title="TRANSACTION COUNT",
        animation_frame='Year',
        animation_group='Quarter',
        height=800
    )

    fig_tra.update_geos(fitbounds="locations", visible=False)
    fig_tra.update_layout(width=900, height=800)
    fig_tra.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig_tra)

def animate_all_amount():
    url = "https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
    response = requests.get(url)
    data1 = json.loads(response.content)
    state_names_tra = [feature['properties']['ST_NM'] for feature in data1['features']]
    state_names_tra.sort()

    df_state_names_tra = pd.DataFrame({'State': state_names_tra})

    frames = []

    for year in map_user['Year'].unique():
        for quarter in Agg_Trans['Quarter'].unique():
            at1 = Agg_Trans[(Agg_Trans['Year'] == year) & (Agg_Trans['Quarter'] == quarter)]
            atf1 = at1[['State', 'Transaction_amount']]
            atf1 = atf1.sort_values(by='State')
            atf1['Year'] = year
            atf1['Quarter'] = quarter
            frames.append(atf1)

    merged_df = pd.concat(frames)

    fig_tra = px.choropleth(
        merged_df, 
        geojson=data1, 
        locations='State', 
        featureidkey='properties.ST_NM', 
        color='Transaction_amount',
        color_continuous_scale='Sunsetdark',
        range_color=(0-1,200000000000),
        hover_name='State',
        title='TRANSACTION AMOUNT',
        animation_frame='Year',
        animation_group='Quarter',
        height=800
    )

    fig_tra.update_geos(fitbounds="locations", visible=False)
    fig_tra.update_layout(width=900, height=800)
    fig_tra.update_layout(title_font=dict(size=33), title_font_color='#801D70',title_x=0.2)
    st.plotly_chart(fig_tra)
    
def Trans_amount(yr):
    year=int(yr)
    at=Agg_Trans[['State','Year','Quarter','Transaction_amount']]
    at1=at.loc[(at['Year']==year)]
    atf1=at1[['State','Transaction_amount']]
    atf1=atf1.sort_values(by='State')

    url = "https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
    response = requests.get(url)
    data1 = json.loads(response.content)
    state_names_tra = [feature['properties']['ST_NM'] for feature in data1['features']]
    state_names_tra.sort()
    df_state_names_tra = pd.DataFrame({'State': state_names_tra})
    merged_df = df_state_names_tra.merge(atf1, on='State')
    merged_df = df_state_names_tra.merge(atf1, on='State')
    merged_df.to_csv('State_trans.csv', index=False)
    df_tra = pd.read_csv('State_trans.csv')
    fig_tra = px.choropleth(
                df_tra,
                         geojson="https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson",
                
        featureidkey='properties.ST_NM',
        locations='State',color='Transaction_amount',
        title='TRANSACTION AMOUNT',
        color_continuous_scale='Sunsetdark',
        range_color=(0,145000000000)
    )
    fig_tra.update_geos(fitbounds="locations", visible=False)
    fig_tra.update_layout(width=900, height=800)
    fig_tra.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig_tra)
    
def Trans_count(yr):
    year=int(yr)
    at=Agg_Trans[['State','Year','Quarter','Transaction_count']]
    at1=at.loc[(at['Year']==year)]
    atf1=at1[['State','Transaction_count']]
    atf1=atf1.sort_values(by='State')
    url = "https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
    response = requests.get(url)
    data1 = json.loads(response.content)
    state_names_tra = [feature['properties']['ST_NM'] for feature in data1['features']]
    state_names_tra.sort()
    df_state_names_tra = pd.DataFrame({'State': state_names_tra})
    merged_df = df_state_names_tra.merge(atf1, on='State')
    merged_df = df_state_names_tra.merge(atf1, on='State')
    merged_df.to_csv('State_trans.csv', index=False)
    df_tra = pd.read_csv('State_trans.csv')
    fig_tra = px.choropleth(
                df_tra,
                         geojson="https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson",
                
        featureidkey='properties.ST_NM',
        locations='State',
        color='Transaction_count',
        title='TRANSACTION COUNT',
        color_continuous_scale='Sunsetdark',
        range_color=(0,250000000)
    )
    fig_tra.update_geos(fitbounds="locations", visible=False)
    fig_tra.update_layout(width=900, height=800)
    fig_tra.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig_tra)
    
def Payment_count():
    attype=Agg_Trans[['Transaction_type','Transaction_count']]
    att1=attype.groupby('Transaction_type')['Transaction_count'].sum()
    att1_df= pd.DataFrame(att1).reset_index()
    fig=px.pie(att1_df, names='Transaction_type',values='Transaction_count',
               title='TRANSACTION TYPE by TRANSACTION COUNT',
               color_discrete_sequence=px.colors.sequential.Sunsetdark,
               hole = 0.4)
    fig.update_layout(width=800, height=700)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70')
    fig.update_traces(hoverinfo='label+percent', textinfo='value', textfont_size=20,
                  marker=dict(line=dict(color='#801D70', width=2)))
    return st.plotly_chart(fig)

def Payment_amount():
    attype=Agg_Trans[['Transaction_type','Transaction_amount']]
    att1=attype.groupby('Transaction_type')['Transaction_amount'].sum()
    att1_df= pd.DataFrame(att1).reset_index()
    fig=px.pie(att1_df,names='Transaction_type',values='Transaction_amount',
                title='TRANSACTION TYPE by TRANSACTION AMOUNT',
                color_discrete_sequence=px.colors.sequential.Sunsetdark,
               hole = 0.4)
    fig.update_layout(width=800, height=700)
    fig.update_layout(title_font=dict(size=33),title_font_color='#801D70')
    fig.update_traces(hoverinfo='label+percent', textinfo='value', textfont_size=20,
                  marker=dict(line=dict(color='#801D70', width=2)))
    return st.plotly_chart(fig)
    
def payment_count_y(year):
    yr=int(year)
    attype=Agg_Trans[['Year','Transaction_type','Transaction_count']]
    att1=attype.loc[(attype['Year']==yr)]

    att1=att1.groupby('Transaction_type')['Transaction_count'].sum()
    att1_df= pd.DataFrame(att1).reset_index()
    fig=px.pie(att1_df,names='Transaction_type',values='Transaction_count', 
                title='TRANSACTION TYPE by TRANSACTION COUNT',
               color_discrete_sequence=px.colors.sequential.Sunsetdark,
               hole = 0.4)
    fig.update_layout(width=800, height=700)
    fig.update_layout(title_font=dict(size=33),title_font_color='#801D70')
    return st.plotly_chart(fig)

def payment_amount_y(year):
    yr=int(year)
    attype=Agg_Trans[['Year','Transaction_type','Transaction_amount']]
    att1=attype.loc[(attype['Year']==yr)]

    att1=att1.groupby('Transaction_type')['Transaction_amount'].sum()
    att1_df= pd.DataFrame(att1).reset_index()
    fig=px.pie(att1_df,names='Transaction_type',values='Transaction_amount', 
                title='TRANSACTION TYPE by TRANSACTION AMOUNT',
               color_discrete_sequence=px.colors.sequential.Sunsetdark,
               hole = 0.4)
    fig.update_layout(width=800, height=700)
    fig.update_layout(title_font=dict(size=33),title_font_color='#801D70')
    return st.plotly_chart(fig)


def reg_state_all(state):
    mu=map_user[['State','District','RegisteredUsers']]
    mu1=mu.loc[(mu['State']==state)]
    mu2=mu1.groupby('District')['RegisteredUsers'].sum()
    bt = pd.DataFrame(mu2).reset_index()
    fig = px.bar(bt, x='District', y='RegisteredUsers',
                 title="REGISTERED USERS IN EACH DISTRICTS",
                 color_discrete_sequence=px.colors.sequential.Sunsetdark)
    fig.update_layout(width=1700, height=700)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70')
    fig.update_traces(marker_color = '#FCDE9C', marker_line_color = '#801D70',
                  marker_line_width = 2)
    return st.plotly_chart(fig)

def reg_state(state,year):
    yr=int(year)
    mu=map_user[['State','Year','District','RegisteredUsers']]
    mu1=mu.loc[(mu['State']==state)&(mu['Year']==yr)]
    mu2=mu1.groupby('District')['RegisteredUsers'].sum()
    bt = pd.DataFrame(mu2).reset_index()
    fig = px.bar(bt, x='District', y='RegisteredUsers',
                 title="REGISTERED USERS IN EACH DISTRICTS",
                 color_discrete_sequence=px.colors.sequential.Sunsetdark)
    fig.update_layout(width=1700, height=700)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70')
    return st.plotly_chart(fig)
    
def ani():
    ag=Agg_Trans
    ag['Year'] = ag['Year'].astype(str)
    ag['Quarter'] = ag['Quarter'].astype(str)
    ag['Year-Q'] = ag.apply(lambda row: '-'.join([row['Year'], row['Quarter']]), axis=1)
    ag.drop(columns=['Year', 'Quarter'], inplace=True)
    
    
#*************************************************Queries********************************************************************************** 

def one():
    au=Agg_user[['Brands','Transaction_Count']]
    brand_transaction_counts = au.groupby('Brands')['Transaction_Count'].sum()
    bt = pd.DataFrame(brand_transaction_counts).reset_index()
    fig= px.pie(bt, values='Transaction_Count', names='Brands',
                 color_discrete_sequence=px.colors.sequential.Sunsetdark, title = 'Top Brands Of Mobiles Used')
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)

def two():
    ag1 = Agg_Trans[['State','Transaction_amount']]
    ag1 = ag1.groupby('State')['Transaction_amount'].sum()
    ag1 = ag1.sort_values()
    agi = ag1.tail(10)
    fig = px.bar(agi, x=agi.index, y='Transaction_amount', title = 'States with Highest Money Trasaction')  
    fig.update_layout(width=900, height=800)
    fig.update_traces(marker_color = '#801D70', marker_line_color = '#FCDE9C', marker_line_width = 2) 
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)
    
def three():
    ag1 = Agg_Trans[['State','Transaction_amount']]
    ag1 = ag1.groupby('State')['Transaction_amount'].sum()
    ag1 = ag1.sort_values()
    agi = ag1.head(10)
    fig = px.bar(agi, x=agi.index, y='Transaction_amount', title = 'States with Lowest Money Trasaction')
    fig.update_layout(width=900, height=800)
    fig.update_traces(marker_color = '#FCDE9C', marker_line_color = '#801D70', marker_line_width = 2)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)
    
def four():
    mt = map_trans[['District', 'Transaction_amount']]
    mt = mt.groupby('District')['Transaction_amount'].sum()
    mt1 = mt.sort_values(ascending=False).head(10).reset_index()
    fig = px.histogram(mt1, x='District', y='Transaction_amount',color='Transaction_amount', title = 'Districts with Highest Money Transactions',color_discrete_sequence=px.colors.sequential.Sunsetdark)
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)
    
def five():
    mt = map_trans[['District', 'Transaction_amount']]
    mt = mt.groupby('District')['Transaction_amount'].sum()
    mt1 = mt.sort_values(ascending=False).tail(10).reset_index()
    fig = px.histogram(mt1, x='District', y='Transaction_amount', color='Transaction_amount', title = 'Districts with Lowest Money Transactions',color_discrete_sequence=px.colors.sequential.Sunsetdark)
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)
    
def six():
    mu=map_user[["State",'AppOpens']]
    mu1=mu.groupby('State')['AppOpens'].sum()
    bt = pd.DataFrame(mu1).reset_index()
    bt=bt.sort_values(by='AppOpens')
    bt1=bt.head(10)
    fig=px.pie(bt1, names='State', values='AppOpens',color_discrete_sequence=px.colors.sequential.Sunsetdark, hole = 0.4, title= 'Top 10 States with AppOpens')
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)
    
def seven():
    mu=map_user[["State",'AppOpens']]
    mu1=mu.groupby('State')['AppOpens'].sum()
    bt = pd.DataFrame(mu1).reset_index()
    bt=bt.sort_values(by='AppOpens')
    bt1=bt.tail(10)
    fig=px.pie(bt1, names='State', values='AppOpens',color_discrete_sequence=px.colors.sequential.Sunsetdark, hole = 0.4, title= 'Least 10 States with AppOpens')
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig) 

def eight():
    ag1 = Agg_Trans[['State','Transaction_count']]
    ag1 = ag1.groupby('State')['Transaction_count'].sum()
    ag1 = ag1.sort_values()
    agi = ag1.head(10)
    fig = px.scatter(agi, x=agi.index, y='Transaction_count',color = 'Transaction_count',color_discrete_sequence=px.colors.sequential.Sunsetdark,size = 'Transaction_count', title = 'States with Highest Trasaction Count')  
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)

def nine():
    ag1 = Agg_Trans[['State','Transaction_count']]
    ag1 = ag1.groupby('State')['Transaction_count'].sum()
    ag1 = ag1.sort_values()
    agi = ag1.tail(10)
    fig = px.scatter(agi, x=agi.index, y='Transaction_count',color = 'Transaction_count',color_discrete_sequence=px.colors.sequential.Sunsetdark,size = 'Transaction_count', title = 'States with Lowest Trasaction Count')  
    fig.update_layout(width=900, height=800)
    fig.update_layout(title_font=dict(size=33), title_font_color='#801D70', title_x=0.2)
    st.plotly_chart(fig)

#**********************************************streamlit part******************************************************************************

st.markdown("<h1 style='text-align: center;color:#ffffff;background: rgb(252,220,155);margin-bottom:-35px;background: linear-gradient(90deg, rgba(252,220,155,1) 0%, rgba(226,74,113,1) 50%, rgba(126,29,111,1) 100%);'>PHONEPE DATA VISUALIZATION AND EXPLORATION</h1>", unsafe_allow_html=True)
st.header('', divider='rainbow')
whitespace = 15
tab_no = ['**Home**','**Explore Data**','**Top Charts**']
tab1, tab2, tab3 = st.tabs([s.center(whitespace,"\u2001") for s in tab_no])
with tab1:
    image = Image.open("logo.png")
    st.image(image,width=150)
    st.markdown("<h2 style='text-align:justify; font-size: 22px;margin-left: 200px;margin-top: -160px;margin-bottom:-200px;line-height:1.5'>PhonePe, founded in December 2015, stands as India's premier digital payment app, revolutionizing financial transactions. By linking bank accounts, credit, and debit cards to its mobile wallet, PhonePe enables convenient digital payments, including groceries, bills, and transfers. As a leader in the sector, it has streamlined banking and finance while enhancing security and accessibility.</h2>", unsafe_allow_html=True)
    st.header('', divider='violet')
    col1,col2=st.columns([3,1])                                  
    with col1:
        st.write('<h1 style="font-size:20px;margin-left:16px;color:#603194; font-size:30px;">SERVICES PROVIDED:</h1>', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:5px;margin-left:1em;font-size:20px">PhonePe is a prominent digital payment platform in India that offers:</p>', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">1. UPI Payments: Send money, pay bills, and make purchases using UPI.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">2. Recharge and Bills: Recharge phones, pay utility bills, and settle expenses.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">3. Money Transfers: Instantly transfer money to contacts and bank accounts.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">4. Online/Offline Payments: Pay at merchants using QR codes.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">5. Investments: Invest in mutual funds and buy gold.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">6. Insurance: Purchase health insurance policies.', unsafe_allow_html=True)         
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">7. Credit Services: Access personal loans and credit cards.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">8. Travel Booking: Book flights, buses, and hotels.', unsafe_allow_html=True)
        st.write('<p style="margin-bottom:0px;margin-left:1em;font-size:20px">9. Gift Cards: Buy and send digital gift cards.', unsafe_allow_html=True)
        st.write('<p style="margin-left:1em;font-size:20px">10. E-commerce: Shop various products and services.', unsafe_allow_html=True)
        
    with col2:   
        with st.container():
            video_path = r"C:\Users\pavit\Desktop\Python\phonepe.mp4"
            st.video(video_path)
    
with tab2:
    tr_year = st.selectbox('**Select Year**', ('All','2018','2019','2020','2021','2022'))
    if tr_year == 'All':
        col1, col2 = st.columns(2)
        with col1:
            animate_all_amount()
            Payment_amount()
           
        with col2:
            animate_all()
            Payment_count()
        state=st.selectbox('**Select State**',('Andaman & Nicobar Islands','Andhra Pradesh', 'Arunachal Pradesh',
       'Assam', 'Bihar', 'Chandigarh', 'Chhattisgarh', 'Dadra & Nagar Haveli & Daman & Diu', 'Delhi', 'Goa', 'Gujarat',
       'Haryana', 'Himachal Pradesh', 'Jammu & Kashmir', 'Jharkhand', 'Karnataka', 'Kerala', 'Ladakh', 'Lakshadweep', 'Madhya Pradesh',
       'Maharashtra', 'Manipur', 'Meghalaya', 'Mizoram', 'Nagaland', 'Odisha', 'Puducherry', 'Punjab', 'Rajasthan', 'Sikkim',
       'Tamil Nadu', 'Telangana', 'Tripura', 'Uttar Pradesh', 'Uttarakhand', 'West Bengal'))
        reg_state_all(state)
        
    else:
        col1, col2 = st.columns(2)
        with col1:
            Trans_amount(tr_year)
            payment_count_y(tr_year)
        with col2:
            Trans_count(tr_year)
            payment_amount_y(tr_year)
        state=st.selectbox('**Select State**',('Andaman & Nicobar Islands','Andhra Pradesh', 'Arunachal Pradesh',
       'Assam', 'Bihar', 'Chandigarh', 'Chhattisgarh', 'Dadra & Nagar Haveli & Daman & Diu', 'Delhi', 'Goa', 'Gujarat',
       'Haryana', 'Himachal Pradesh', 'Jammu & Kashmir', 'Jharkhand', 'Karnataka', 'Kerala', 'Ladakh', 'Lakshadweep', 'Madhya Pradesh',
       'Maharashtra', 'Manipur', 'Meghalaya', 'Mizoram', 'Nagaland', 'Odisha', 'Puducherry', 'Punjab', 'Rajasthan', 'Sikkim',
       'Tamil Nadu', 'Telangana', 'Tripura', 'Uttar Pradesh', 'Uttarakhand', 'West Bengal'))
        reg_state(state,tr_year)
        
with tab3:
    q=st.selectbox('**Select**', ('Top Brands Of Mobiles Used','States with Highest and Lowest Money Trasactions','Districts with Highest and Lowest Money Transactions','Top and Least 10 States with AppOpens','States with Highest and Lowest Trasaction Count'))
    if q=='Top Brands Of Mobiles Used':
        one()
    elif q=='States with Highest and Lowest Money Trasactions':
        col1, col2 = st.columns(2)
        with col1:
            two()
        with col2:
            three()
    elif q=='Districts with Highest and Lowest Money Transactions':
        col1, col2 = st.columns(2)
        with col1:
            four()
        with col2:
            five()
    elif q=='Top and Least 10 States with AppOpens':
        col1, col2 = st.columns(2)
        with col1:
            six()
        with col2:
            seven()
    elif q=='States with Highest and Lowest Trasaction Count':
        col1, col2 = st.columns(2)
        with col1:
            eight()
        with col2:
            nine()
