# Dasboard-Iklim
Dasboard ini berisi data prediksi ONI, Curah Hujan, Suhu, dan Kelembaban di daerah kawasan Gunung Merapi dan dataran rendah sekitarnya di Sleman

# ==========================================================
# DASHBOARD PREDIKSI CURAH HUJAN BERDASARKAN ONI
# Kabupaten Sleman (1995–2030)
# ==========================================================

import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots

st.set_page_config(
    page_title="Dashboard Prediksi Curah Hujan",
    page_icon="🌧️",
    layout="wide",
    initial_sidebar_state="expanded"
)

st.markdown("""
<style>

.main{
    background-color:#f5f7fb;
}

h1{
    color:#003366;
    text-align:center;
}

h2{
    color:#003366;
}

div[data-testid="metric-container"]{
    background:white;
    border-radius:15px;
    padding:20px;
    box-shadow:0px 4px 10px rgba(0,0,0,0.15);
}

section[data-testid="stSidebar"]{
    background:#0F4C81;
}

section[data-testid="stSidebar"] *{
    color:white;
}

</style>
""",unsafe_allow_html=True)

st.markdown(
"""
# 🌧️ Dashboard Prediksi Curah Hujan

### Berdasarkan Oceanic Niño Index (ONI)

#### Kabupaten Sleman (1995–2030)

---
"""
)

df = pd.read_excel(
    "C:/Users/USER/Downloads/data keseluruhan 1994-2025 (2).xlsx"
)
# Rapikan nama kolom
df.columns = df.columns.str.strip().str.lower()
df["kecamatan"] = (
    df["kecamatan"]
    .astype(str)
    .str.strip()
    .str.title()
)
#masukin ga ya?
#st.write("Jumlah Data :",len(df))
#st.write(df.head())

def kategori_oni(x):

    if x >= 2:
        return "El Nino Sangat Kuat"

    elif x >= 1.5:
        return "El Nino Kuat"

    elif x >= 1:
        return "El Nino Moderat"

    elif x >= 0.5:
        return "El Nino Lemah"

    elif -0.5 < x < 0.5:
        return "Netral"

    elif -0.9 <= x <= -0.5:
        return "La Nina Lemah"

    elif -1.4 <= x < -0.9:
        return "La Nina Moderat"

    elif -1.9 <= x < -1.4:
        return "La Nina Kuat"

    else:
        return "La Nina Sangat Kuat"
    
# Membuat kolom kategori ONI
df["kategori_oni"] = df["oni"].apply(kategori_oni)

# Membuat kolom jenis data
df["jenis_data"] = np.where(
    df["tahun"] <= 2024,
    "Historis",
    "Forecasting"
)
st.sidebar.title("📂 FILTER DATA")

# Pilih Historis atau Forecasting
jenis = st.sidebar.radio(
    "Jenis Data",
    ["Historis", "Forecasting"]
)

# Daftar tahun mengikuti jenis data
if jenis == "Historis":
    daftar_tahun = sorted(
        df[df["jenis_data"] == "Historis"]["tahun"].unique()
    )
else:
    daftar_tahun = sorted(
        df[df["jenis_data"] == "Forecasting"]["tahun"].unique()
    )

tahun = st.sidebar.selectbox(
    "📅 Tahun",
    daftar_tahun
)

data = df[
    (df["jenis_data"] == jenis) &
    (df["tahun"] == tahun)
].copy()

st.subheader(f"📋 Data Tahun {tahun}")

st.dataframe(
    data[
        [
            "tahun",
            "kecamatan",
            "periode",
            "oni",
            "kategori_oni",
            "curah",
            "suhu",
            "rh"
        ]
    ],
    use_container_width=True,
    hide_index=True
)

st.markdown("---")
st.subheader("🌧 Curah Hujan Setiap Kecamatan")

fig_curah = px.line(
    data,
    x="periode",
    y="curah",
    color="kecamatan",
    markers=True,
    line_group="kecamatan",
    title=f"Curah Hujan Tahun {tahun}"
)

fig_curah.update_layout(
    template="plotly_white",
    xaxis_title="Periode",
    yaxis_title="Curah Hujan (mm)"
)

st.plotly_chart(fig_curah, use_container_width=True)

st.markdown("### 🏞 Perbandingan Wilayah Curah Hujan")

col1, col2 = st.columns(2)

# ===============================
# DATARAN RENDAH
# ===============================
with col1:

    data_rendah = data[
        data["kecamatan"].isin(["Depok","Moyudan"])
    ]

    fig_rendah = px.line(
        data_rendah,
        x="periode",
        y="curah",
        color="kecamatan",
        markers=True,
        title="🌾 Dataran Rendah (Depok & Moyudan)"
    )

    fig_rendah.update_layout(
        template="plotly_white",
        height=400
    )

    st.plotly_chart(fig_rendah,use_container_width=True)

# ===============================
# LERENG MERAPI
# ===============================
with col2:

    data_lereng = data[
        data["kecamatan"].isin(["Tempel","Cangkringan"])
    ]

    fig_lereng = px.line(
        data_lereng,
        x="periode",
        y="curah",
        color="kecamatan",
        markers=True,
        title="⛰ Lereng Merapi (Tempel & Cangkringan)"
    )

    fig_lereng.update_layout(
        template="plotly_white",
        height=400
    )

    st.plotly_chart(fig_lereng,use_container_width=True)
    
st.markdown("---")
st.subheader("🌡 Suhu Udara Setiap Kecamatan")

fig_suhu = px.line(
    data,
    x="periode",
    y="suhu",
    color="kecamatan",
    markers=True,
    title=f"Suhu Tahun {tahun}"
)

fig_suhu.update_layout(
    template="plotly_white",
    xaxis_title="Periode",
    yaxis_title="Suhu (°C)"
)

st.plotly_chart(fig_suhu, use_container_width=True)

st.markdown("---")
st.subheader("💧 Kelembapan Udara")

fig_rh = px.line(
    data,
    x="periode",
    y="rh",
    color="kecamatan",
    markers=True,
    title=f"Kelembapan Tahun {tahun}"
)

fig_rh.update_layout(
    template="plotly_white",
    xaxis_title="Periode",
    yaxis_title="RH (%)"
)

st.plotly_chart(fig_rh, use_container_width=True)

st.markdown("---")
st.subheader("🌊 Oceanic Niño Index (ONI)")

oni_data = data.drop_duplicates(subset="periode")

fig_oni = px.line(
    oni_data,
    x="periode",
    y="oni",
    markers=True,
    title=f"Nilai ONI Tahun {tahun}"
)

fig_oni.add_hline(y=0, line_dash="dash")

fig_oni.update_layout(
    template="plotly_white",
    xaxis_title="Periode",
    yaxis_title="ONI"
)

st.plotly_chart(fig_oni, use_container_width=True)



