import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go


# ============================================================
# DERMAFORM
# SMART SKINCARE FORMULATION LAB
# ============================================================
# College project:
# Application of Chemistry in Cosmetic & Skincare Formulation
#
# IMPORTANT UI FIX:
# All custom visual HTML is rendered with st.html().
# This prevents raw <div>, <br>, and CSS markup from appearing
# as visible code inside the application.
# ============================================================


# ============================================================
# PAGE CONFIGURATION
# ============================================================

st.set_page_config(
    page_title="DERMAFORM | Smart Skincare Formulation Lab",
    page_icon="🧪",
    layout="wide",
    initial_sidebar_state="expanded",
)


# ============================================================
# SESSION STATE
# ============================================================

DEFAULT_STATE = {
    "page": "🏠 Home",
    "formula": [],
    "formula_name": "Hydrating Botanical Serum",
    "product_type": "Serum",
    "batch_size": 100.0,
    "target_ph": 5.5,
    "saved_formulas": 0,
}

for key, value in DEFAULT_STATE.items():
    if key not in st.session_state:
        st.session_state[key] = value


# ============================================================
# GLOBAL CSS
# ============================================================
# We intentionally use darker text than the earlier version.
# This fixes low-contrast text on ivory/pastel backgrounds.
# ============================================================

st.html(
    """
    <style>
    :root {
        --ivory: #F7F2EC;
        --cream: #FFFDF9;
        --white: #FFFFFF;
        --blush: #E7C9CB;
        --blush-dark: #8C6268;
        --peach: #F2D9CB;
        --sage: #C7D4C1;
        --sage-dark: #64745E;
        --lavender: #DDD4E6;
        --beige: #E4D3C1;
        --brown: #332B27;
        --brown-2: #4F443E;
        --muted: #625750;
        --border: rgba(65, 53, 46, 0.10);
        --shadow: 0 16px 45px rgba(70, 55, 47, 0.09);
    }

    html, body, [class*="css"] {
        font-family: Arial, Helvetica, sans-serif;
    }

    /* ========================================================
       ACCESSIBLE TEXT OVERRIDES
       Keep every label readable on the ivory skincare palette.
       ======================================================== */
    section[data-testid="stSidebar"] {
        background: #FFFCF8 !important;
        color: #2F2824 !important;
    }

    section[data-testid="stSidebar"] * {
        color: #3B312C !important;
    }

    section[data-testid="stSidebar"] .sidebar-icon,
    section[data-testid="stSidebar"] .sidebar-icon * {
        color: initial !important;
    }

    section[data-testid="stSidebar"] [data-testid="stRadio"] label {
        color: #2F2824 !important;
        opacity: 1 !important;
        font-weight: 600 !important;
    }

    section[data-testid="stSidebar"] [data-testid="stRadio"] label p,
    section[data-testid="stSidebar"] [data-testid="stRadio"] label span {
        color: #2F2824 !important;
        opacity: 1 !important;
    }

    section[data-testid="stSidebar"] [data-testid="stRadio"] [role="radiogroup"] {
        gap: 0.25rem !important;
    }

    section[data-testid="stSidebar"] .nav-caption {
        color: #55443C !important;
        opacity: 1 !important;
    }

    section[data-testid="stSidebar"] .sidebar-subtitle {
        color: #65544B !important;
        opacity: 1 !important;
    }

    section[data-testid="stSidebar"] .sidebar-note,
    section[data-testid="stSidebar"] .sidebar-note * {
        color: #3D332E !important;
        opacity: 1 !important;
    }

    /* Streamlit widgets */
    [data-testid="stWidgetLabel"],
    [data-testid="stWidgetLabel"] p,
    [data-testid="stMarkdownContainer"],
    [data-testid="stMarkdownContainer"] p,
    [data-testid="stMarkdownContainer"] li {
        color: #332B27 !important;
        opacity: 1 !important;
    }

    input, textarea, select {
        color: #2F2824 !important;
        background: #FFFFFF !important;
    }

    .stApp {
        background:
            radial-gradient(
                circle at 5% 5%,
                rgba(231, 201, 203, 0.34),
                transparent 24%
            ),
            radial-gradient(
                circle at 94% 12%,
                rgba(199, 212, 193, 0.30),
                transparent 24%
            ),
            linear-gradient(
                135deg,
                #F7F2EC 0%,
                #FFFDF9 52%,
                #F7EFEB 100%
            );
        color: var(--brown);
    }

    .block-container {
        max-width: 1450px;
        padding-top: 2rem;
        padding-bottom: 4rem;
    }

    section[data-testid="stSidebar"] {
        background: rgba(255, 253, 249, 0.98);
        border-right: 1px solid var(--border);
    }

    section[data-testid="stSidebar"] .block-container {
        padding-top: 1.4rem;
    }

    .sidebar-brand {
        text-align: center;
        padding: 8px 4px 18px;
    }

    .sidebar-icon {
        font-size: 42px;
        line-height: 1;
        margin-bottom: 8px;
    }

    .sidebar-title {
        font-family: Georgia, "Times New Roman", serif;
        font-size: 28px;
        font-weight: 600;
        letter-spacing: 0.5px;
        color: var(--brown);
    }

    .sidebar-subtitle {
        font-size: 10px;
        letter-spacing: 2px;
        color: var(--muted);
        margin-top: 5px;
    }

    .nav-caption {
        font-size: 10px;
        font-weight: 700;
        letter-spacing: 1.8px;
        color: #756860;
        margin: 14px 0 8px;
    }

    .sidebar-note {
        background: linear-gradient(
            135deg,
            rgba(231, 201, 203, 0.62),
            rgba(199, 212, 193, 0.40)
        );
        border: 1px solid rgba(255,255,255,0.8);
        border-radius: 18px;
        padding: 16px;
        color: var(--brown-2);
        font-size: 11px;
        line-height: 1.65;
        margin-top: 20px;
        box-shadow: 0 10px 28px rgba(70,55,47,0.05);
    }

    .page-title {
        font-family: Georgia, "Times New Roman", serif;
        font-size: 44px;
        font-weight: 600;
        color: var(--brown);
        line-height: 1.1;
        margin-bottom: 6px;
    }

    .page-subtitle {
        color: var(--muted);
        font-size: 15px;
        line-height: 1.6;
        margin-bottom: 25px;
    }

    .hero-shell {
        position: relative;
        overflow: hidden;
        min-height: 440px;
        border-radius: 34px;
        padding: 50px 52px;
        background:
            radial-gradient(
                circle at 88% 20%,
                rgba(221,212,230,0.65),
                transparent 22%
            ),
            linear-gradient(
                135deg,
                rgba(255,255,255,0.88),
                rgba(251,241,237,0.88)
            );
        border: 1px solid rgba(255,255,255,0.96);
        box-shadow: var(--shadow);
    }

    .hero-kicker {
        display: inline-block;
        padding: 8px 14px;
        border-radius: 999px;
        background: rgba(231,201,203,0.42);
        color: var(--brown-2);
        font-size: 11px;
        letter-spacing: 1.4px;
        font-weight: 700;
        text-transform: uppercase;
    }

    .hero-title {
        font-family: Georgia, "Times New Roman", serif;
        font-size: clamp(45px, 6vw, 76px);
        font-weight: 600;
        line-height: 0.98;
        color: var(--brown);
        margin-top: 20px;
    }

    .hero-title span {
        color: var(--blush-dark);
    }

    .hero-tagline {
        color: var(--brown-2);
        font-size: 20px;
        font-weight: 600;
        margin-top: 17px;
    }

    .hero-description {
        color: #554A44;
        max-width: 650px;
        font-size: 14px;
        line-height: 1.8;
        margin-top: 16px;
    }

    .hero-pill-row {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        margin-top: 22px;
    }

    .hero-pill {
        padding: 8px 12px;
        border-radius: 999px;
        background: rgba(255,255,255,0.76);
        border: 1px solid rgba(80,60,50,0.07);
        color: #5D514A;
        font-size: 11px;
    }

    .visual-stage {
        position: relative;
        height: 350px;
        width: 100%;
    }

    .visual-shadow {
        position: absolute;
        left: 18%;
        right: 10%;
        bottom: 22px;
        height: 30px;
        border-radius: 50%;
        background: rgba(70,55,47,0.10);
        filter: blur(10px);
    }

    .serum-bottle {
        position: absolute;
        width: 118px;
        height: 210px;
        left: 34%;
        top: 85px;
        border-radius: 22px 22px 34px 34px;
        background:
            linear-gradient(
                105deg,
                rgba(218,184,181,0.95),
                rgba(255,242,236,0.98)
            );
        border: 1px solid rgba(255,255,255,0.75);
        box-shadow:
            0 28px 48px rgba(70,55,47,0.17),
            inset -12px 0 20px rgba(255,255,255,0.35);
        transform: rotate(-5deg);
        z-index: 4;
    }

    .serum-bottle::before {
        content: "";
        position: absolute;
        width: 60px;
        height: 54px;
        left: 29px;
        top: -50px;
        border-radius: 8px 8px 4px 4px;
        background: linear-gradient(
            90deg,
            #9E8E86,
            #D7C8C0,
            #9E8E86
        );
        box-shadow: 0 -8px 0 rgba(90,70,62,0.12);
    }

    .serum-label {
        position: absolute;
        left: 12px;
        right: 12px;
        top: 86px;
        padding: 14px 5px;
        border-radius: 8px;
        text-align: center;
        background: rgba(255,253,249,0.83);
        color: #554A44;
        font-family: Georgia, "Times New Roman", serif;
        font-size: 10px;
        letter-spacing: 1px;
    }

    .cream-jar {
        position: absolute;
        width: 150px;
        height: 92px;
        left: 52%;
        top: 208px;
        border-radius: 17px 17px 36px 36px;
        background: linear-gradient(
            145deg,
            #E7D4C5,
            #FFF8F1
        );
        box-shadow: 0 24px 42px rgba(70,55,47,0.14);
        transform: rotate(5deg);
        z-index: 5;
    }

    .cream-jar::before {
        content: "";
        position: absolute;
        width: 160px;
        height: 27px;
        left: -5px;
        top: -20px;
        border-radius: 8px;
        background: linear-gradient(
            90deg,
            #B5A59C,
            #DED3CC,
            #A99991
        );
    }

    .cream-label {
        position: absolute;
        top: 36px;
        left: 20px;
        right: 20px;
        text-align: center;
        color: #5B4F49;
        font-size: 10px;
        letter-spacing: 1px;
    }

    .leaf {
        position: absolute;
        width: 80px;
        height: 40px;
        border-radius: 100% 0 100% 0;
        background: var(--sage);
        opacity: 0.72;
        z-index: 2;
    }

    .leaf-a {
        right: 5%;
        top: 42px;
        transform: rotate(28deg);
    }

    .leaf-b {
        left: 5%;
        bottom: 60px;
        transform: rotate(-30deg);
    }

    .leaf-c {
        right: 12%;
        bottom: 28px;
        transform: rotate(48deg) scale(0.72);
        background: var(--sage-dark);
        opacity: 0.28;
    }

    .drop {
        position: absolute;
        width: 24px;
        height: 35px;
        border-radius: 70% 30% 65% 35%;
        background: #D1E3E1;
        opacity: 0.82;
        z-index: 6;
    }

    .drop-a {
        left: 14%;
        top: 60px;
        transform: rotate(22deg);
    }

    .drop-b {
        right: 13%;
        bottom: 100px;
        transform: rotate(-25deg);
    }

    .drop-c {
        left: 20%;
        top: 230px;
        transform: rotate(12deg) scale(0.62);
    }

    .molecule {
        position: absolute;
        color: rgba(105, 90, 80, 0.17);
        font-size: 29px;
        line-height: 1.05;
        z-index: 1;
        user-select: none;
    }

    .molecule-a {
        top: 8px;
        right: 10px;
    }

    .molecule-b {
        bottom: 5px;
        left: 10px;
        font-size: 24px;
    }

    .soft-card {
        background: rgba(255,255,255,0.84);
        border: 1px solid rgba(255,255,255,0.96);
        border-radius: 22px;
        padding: 23px;
        box-shadow: 0 12px 34px rgba(70,55,47,0.065);
    }

    .feature-card {
        min-height: 170px;
        background: rgba(255,255,255,0.84);
        border: 1px solid rgba(255,255,255,0.96);
        border-radius: 22px;
        padding: 24px;
        box-shadow: 0 12px 34px rgba(70,55,47,0.065);
    }

    .feature-icon {
        font-size: 30px;
        margin-bottom: 10px;
    }

    .feature-title {
        color: var(--brown);
        font-family: Georgia, "Times New Roman", serif;
        font-size: 21px;
        font-weight: 600;
    }

    .feature-text {
        color: #625750;
        font-size: 13px;
        line-height: 1.65;
        margin-top: 8px;
    }

    .metric-card {
        background: rgba(255,255,255,0.86);
        border: 1px solid rgba(255,255,255,0.96);
        border-radius: 20px;
        padding: 19px;
        box-shadow: 0 10px 28px rgba(70,55,47,0.06);
        min-height: 120px;
    }

    .metric-icon {
        font-size: 25px;
    }

    .metric-value {
        color: var(--brown);
        font-family: Georgia, "Times New Roman", serif;
        font-size: 29px;
        font-weight: 600;
        margin-top: 5px;
    }

    .metric-label {
        color: #625750;
        font-size: 11px;
        margin-top: 3px;
    }

    .ingredient-card {
        background: rgba(255,255,255,0.88);
        border: 1px solid rgba(70,55,47,0.07);
        border-radius: 17px;
        padding: 17px;
        margin-bottom: 10px;
    }

    .ingredient-name {
        color: var(--brown);
        font-family: Georgia, "Times New Roman", serif;
        font-size: 19px;
        font-weight: 600;
    }

    .ingredient-meta {
        color: #625750;
        font-size: 12px;
        line-height: 1.6;
        margin-top: 5px;
    }

    .status {
        border-radius: 15px;
        padding: 14px 17px;
        font-weight: 700;
        font-size: 13px;
    }

    .status-green {
        background: #DCE8D8;
        color: #476242;
    }

    .status-yellow {
        background: #F3E7C5;
        color: #755E2C;
    }

    .status-red {
        background: #F0D8D5;
        color: #824B46;
    }

    .report-header {
        border-radius: 25px;
        padding: 28px;
        background: linear-gradient(
            135deg,
            rgba(255,255,255,0.92),
            rgba(244,226,219,0.75)
        );
        border: 1px solid rgba(255,255,255,0.95);
        box-shadow: 0 15px 40px rgba(70,55,47,0.08);
    }

    .report-product {
        font-family: Georgia, "Times New Roman", serif;
        color: var(--brown);
        font-size: 32px;
        font-weight: 600;
    }

    .report-meta {
        color: #625750;
        font-size: 13px;
        margin-top: 7px;
    }

    .disclaimer {
        background: rgba(237, 229, 217, 0.72);
        border: 1px solid rgba(100,80,65,0.08);
        border-radius: 15px;
        padding: 14px 17px;
        color: #5E514A;
        font-size: 12px;
        line-height: 1.65;
    }

    .footer {
        text-align: center;
        color: #756961;
        font-size: 11px;
        line-height: 1.6;
        margin-top: 50px;
        padding: 25px 10px;
    }

    #MainMenu {
        visibility: hidden;
    }

    footer {
        visibility: hidden;
    }

    header {
        background: transparent !important;
    }

    div[data-testid="stMetric"] {
        background: rgba(255,255,255,0.80);
        border-radius: 17px;
        padding: 12px;
    }

    .stButton > button {
        min-height: 43px;
        border-radius: 13px;
        border: 1px solid rgba(70,55,47,0.10);
        background: #FFFDF9;
        color: #3D332E;
        font-weight: 600;
    }

    .stButton > button:hover {
        background: #F3E2DE;
        border-color: #C8A8A8;
        color: #332B27;
    }

    .stDownloadButton > button {
        border-radius: 13px;
    }

    .stTextInput input,
    .stNumberInput input,
    div[data-baseweb="select"] {
        border-radius: 12px !important;
    }

    </style>
    """
)


# ============================================================
# INGREDIENT DATABASE
# ============================================================

INGREDIENTS = {
    "Purified Water": {
        "inci": "Aqua",
        "function": "Solvent / vehicle",
        "role": "Base",
        "phase": "Water Phase",
        "solubility": "Universal solvent",
        "use": "q.s. to 100%",
        "ph": "Formulation dependent",
        "category": "Base",
        "icon": "💦",
    },
    "Glycerin": {
        "inci": "Glycerin",
        "function": "Humectant",
        "role": "Hydration",
        "phase": "Water Phase",
        "solubility": "Water soluble",
        "use": "1–10%",
        "ph": "Broad compatibility",
        "category": "Hydrators",
        "icon": "💧",
    },
    "Hyaluronic Acid": {
        "inci": "Sodium Hyaluronate",
        "function": "Humectant / film former",
        "role": "Hydration",
        "phase": "Water Phase",
        "solubility": "Water dispersible",
        "use": "0.1–2%",
        "ph": "Approx. pH 4–8",
        "category": "Hydrators",
        "icon": "💧",
    },
    "Aloe Vera": {
        "inci": "Aloe Barbadensis Leaf Juice",
        "function": "Skin conditioning",
        "role": "Soothing / hydration",
        "phase": "Water Phase",
        "solubility": "Water soluble",
        "use": "5–100%",
        "ph": "Approx. pH 3.5–5.5",
        "category": "Botanicals",
        "icon": "🌿",
    },
    "Green Tea Extract": {
        "inci": "Camellia Sinensis Leaf Extract",
        "function": "Antioxidant",
        "role": "Antioxidant skincare",
        "phase": "Water Phase",
        "solubility": "Usually water soluble",
        "use": "0.5–5%",
        "ph": "Formulation dependent",
        "category": "Botanicals",
        "icon": "🍃",
    },
    "Oat Extract": {
        "inci": "Avena Sativa Kernel Extract",
        "function": "Skin conditioning",
        "role": "Soothing",
        "phase": "Water Phase",
        "solubility": "Usually water soluble",
        "use": "0.5–5%",
        "ph": "Formulation dependent",
        "category": "Botanicals",
        "icon": "🌾",
    },
    "Niacinamide": {
        "inci": "Niacinamide",
        "function": "Skin conditioning",
        "role": "Barrier support",
        "phase": "Water Phase",
        "solubility": "Water soluble",
        "use": "2–5%",
        "ph": "Approx. pH 5–7",
        "category": "Actives",
        "icon": "✨",
    },
    "Panthenol": {
        "inci": "Panthenol",
        "function": "Humectant",
        "role": "Conditioning / hydration",
        "phase": "Water Phase",
        "solubility": "Water soluble",
        "use": "0.5–5%",
        "ph": "Approx. pH 4–7",
        "category": "Hydrators",
        "icon": "💧",
    },
    "Vitamin E": {
        "inci": "Tocopherol",
        "function": "Antioxidant",
        "role": "Oil-phase antioxidant",
        "phase": "Oil Phase",
        "solubility": "Oil soluble",
        "use": "0.1–1%",
        "ph": "Not primarily pH dependent",
        "category": "Actives",
        "icon": "✨",
    },
    "Shea Butter": {
        "inci": "Butyrospermum Parkii Butter",
        "function": "Emollient",
        "role": "Skin conditioning",
        "phase": "Oil Phase",
        "solubility": "Oil soluble",
        "use": "1–20%",
        "ph": "Not primarily pH dependent",
        "category": "Emollients",
        "icon": "🧴",
    },
    "Jojoba Oil": {
        "inci": "Simmondsia Chinensis Seed Oil",
        "function": "Emollient",
        "role": "Skin conditioning",
        "phase": "Oil Phase",
        "solubility": "Oil soluble",
        "use": "1–30%",
        "ph": "Not primarily pH dependent",
        "category": "Emollients",
        "icon": "🌿",
    },
    "Coco Glucoside": {
        "inci": "Coco-Glucoside",
        "function": "Surfactant",
        "role": "Cleansing",
        "phase": "Surfactant Phase",
        "solubility": "Water dispersible",
        "use": "5–30%",
        "ph": "Formulation dependent",
        "category": "Surfactants",
        "icon": "🫧",
    },
    "Xanthan Gum": {
        "inci": "Xanthan Gum",
        "function": "Thickener",
        "role": "Viscosity control",
        "phase": "Water Phase",
        "solubility": "Dispersible in water",
        "use": "0.1–1%",
        "ph": "Broad range",
        "category": "Thickeners",
        "icon": "🧪",
    },
    "Citric Acid": {
        "inci": "Citric Acid",
        "function": "pH adjuster",
        "role": "pH adjustment",
        "phase": "Water Phase",
        "solubility": "Water soluble",
        "use": "As required",
        "ph": "Acidic",
        "category": "pH Adjusters",
        "icon": "⚗️",
    },
}


# ============================================================
# FORMULATION DEFAULTS
# ============================================================

FORMULA_PRESETS = {
    "Hydrating Serum": [
        ("Purified Water", 90.0, "Water Phase"),
        ("Glycerin", 5.0, "Water Phase"),
        ("Aloe Vera", 3.0, "Water Phase"),
        ("Panthenol", 1.0, "Water Phase"),
        ("Hyaluronic Acid", 1.0, "Water Phase"),
    ],
    "Botanical Serum": [
        ("Purified Water", 91.0, "Water Phase"),
        ("Glycerin", 4.0, "Water Phase"),
        ("Aloe Vera", 2.0, "Water Phase"),
        ("Green Tea Extract", 2.0, "Water Phase"),
        ("Panthenol", 1.0, "Water Phase"),
    ],
    "Cleansing Base": [
        ("Purified Water", 65.0, "Water Phase"),
        ("Coco Glucoside", 25.0, "Surfactant Phase"),
        ("Aloe Vera", 5.0, "Water Phase"),
        ("Glycerin", 4.0, "Water Phase"),
        ("Xanthan Gum", 1.0, "Water Phase"),
    ],
}


# ============================================================
# HELPER FUNCTIONS
# ============================================================

def calculate_quantity(percentage, batch_size):
    return (percentage / 100.0) * batch_size


def formula_total():
    return sum(item["percentage"] for item in st.session_state.formula)


def remaining_percentage():
    return 100.0 - formula_total()


def formula_dataframe():
    rows = []
    for item in st.session_state.formula:
        rows.append(
            {
                "Ingredient": item["name"],
                "INCI Name": item["inci"],
                "Function": item["function"],
                "Phase": item["phase"],
                "Percentage (%)": round(item["percentage"], 3),
                "Quantity (g)": round(
                    calculate_quantity(
                        item["percentage"],
                        st.session_state.batch_size,
                    ),
                    3,
                ),
            }
        )
    return pd.DataFrame(rows)


def add_ingredient(name, percentage, phase):
    if percentage <= 0:
        return False, "Percentage must be greater than zero."

    if formula_total() + percentage > 100:
        return False, "The total formulation cannot exceed 100%."

    data = INGREDIENTS[name]

    st.session_state.formula.append(
        {
            "name": name,
            "inci": data["inci"],
            "function": data["function"],
            "category": data["category"],
            "phase": phase,
            "percentage": float(percentage),
            "quantity": calculate_quantity(
                percentage,
                st.session_state.batch_size,
            ),
        }
    )

    return True, f"{name} added to the formula."


def load_preset(preset_name):
    st.session_state.formula = []

    for name, percentage, phase in FORMULA_PRESETS[preset_name]:
        data = INGREDIENTS[name]
        st.session_state.formula.append(
            {
                "name": name,
                "inci": data["inci"],
                "function": data["function"],
                "category": data["category"],
                "phase": phase,
                "percentage": percentage,
                "quantity": calculate_quantity(
                    percentage,
                    st.session_state.batch_size,
                ),
            }
        )


def stability_score():
    score = 70.0
    total = formula_total()

    if len(st.session_state.formula) >= 3:
        score += 8

    if abs(total - 100.0) < 0.001:
        score += 10
    elif total > 100:
        score -= 30
    else:
        score -= 5

    phases = {item["phase"] for item in st.session_state.formula}

    if len(phases) <= 2:
        score += 5

    if any(
        item["name"] == "Citric Acid"
        for item in st.session_state.formula
    ):
        score -= 2

    return int(max(0, min(100, score)))


def compatibility_pair(a, b):
    if a == b:
        return (
            "Review Required",
            "yellow",
            "The same ingredient was selected twice. Review the formula before proceeding.",
        )

    pair = frozenset([a, b])

    known_compatible = {
        frozenset(["Glycerin", "Hyaluronic Acid"]),
        frozenset(["Glycerin", "Aloe Vera"]),
        frozenset(["Glycerin", "Panthenol"]),
        frozenset(["Aloe Vera", "Panthenol"]),
        frozenset(["Green Tea Extract", "Glycerin"]),
        frozenset(["Vitamin E", "Jojoba Oil"]),
        frozenset(["Shea Butter", "Vitamin E"]),
    }

    review_pairs = {
        frozenset(["Niacinamide", "Citric Acid"]),
        frozenset(["Vitamin E", "Glycerin"]),
        frozenset(["Coco Glucoside", "Citric Acid"]),
    }

    if pair in known_compatible:
        return (
            "Compatible",
            "green",
            "No basic conflict is flagged by this educational screening rule.",
        )

    if pair in review_pairs:
        return (
            "Review Required",
            "yellow",
            "Review concentration, pH, processing conditions and supplier technical guidance.",
        )

    return (
        "Review Required",
        "yellow",
        "No simple rule is being applied here. Review the ingredients using technical documentation and laboratory testing.",
    )


def category_icon(category):
    icons = {
        "Hydrators": "💧",
        "Botanicals": "🌿",
        "Actives": "✨",
        "Emollients": "🧴",
        "Surfactants": "🫧",
        "pH Adjusters": "⚗️",
        "Thickeners": "🧪",
        "Base": "💦",
    }
    return icons.get(category, "🧪")


def phase_distribution():
    result = {}
    for item in st.session_state.formula:
        phase = item["phase"]
        result[phase] = result.get(phase, 0) + item["percentage"]
    return result


def ingredient_category_distribution():
    result = {}
    for item in st.session_state.formula:
        category = item["category"]
        result[category] = result.get(category, 0) + item["percentage"]
    return result


# ============================================================
# SIDEBAR
# ============================================================

with st.sidebar:
    st.html(
        """
        <div class="sidebar-brand">
            <div class="sidebar-icon">🧪</div>
            <div class="sidebar-title">DERMAFORM</div>
            <div class="sidebar-subtitle">SMART SKINCARE LAB</div>
        </div>
        """
    )

    st.markdown("---")

    st.html(
        '<div class="nav-caption">EXPLORE LAB</div>'
    )

    navigation = [
        "🏠 Home",
        "📊 Dashboard",
        "🧴 Formulation Lab",
        "🌿 Ingredient Library",
        "⚗️ Chemistry Tools",
        "🧬 Compatibility",
        "📈 Stability",
        "📄 Formula Report",
    ]

    current_index = (
        navigation.index(st.session_state.page)
        if st.session_state.page in navigation
        else 0
    )

    selected = st.radio(
        "Lab navigation",
        navigation,
        index=current_index,
        label_visibility="collapsed",
    )

    if selected != st.session_state.page:
        st.session_state.page = selected
        st.rerun()

    st.html(
        """
        <div class="sidebar-note">
            <b>🎓 COLLEGE PROJECT</b><br><br>
            Application of Chemistry in Cosmetic &amp; Skincare Formulation.
            <br><br>
            <b>Educational platform</b><br>
            Designed for formulation learning, calculations and live academic demonstration.
        </div>
        """
    )


# ============================================================
# HOME PAGE
# ============================================================

if st.session_state.page == "🏠 Home":
    st.html(
        """
        <div class="hero-shell">
            <div class="hero-kicker">🧪 Cosmetic Chemistry • Digital Lab</div>

            <div class="hero-title">
                DERMA<span>FORM</span>
            </div>

            <div class="hero-tagline">
                Smart Skincare Formulation Lab
            </div>

            <div class="hero-description">
                <b>Where Chemistry Meets Skincare.</b><br><br>
                Explore the chemistry behind skincare formulation —
                from ingredient selection and concentration calculations
                to compatibility, pH and stability analysis.
            </div>

            <div class="hero-pill-row">
                <div class="hero-pill">🧴 Build</div>
                <div class="hero-pill">💧 Calculate</div>
                <div class="hero-pill">🌿 Explore</div>
                <div class="hero-pill">🧬 Analyse</div>
                <div class="hero-pill">📊 Understand</div>
            </div>

            <div class="visual-stage">
                <div class="visual-shadow"></div>

                <div class="molecule molecule-a">
                    ◇──○<br>
                    │ ╲<br>
                    ○──◇
                </div>

                <div class="molecule molecule-b">
                    ○──◇──○
                </div>

                <div class="leaf leaf-a"></div>
                <div class="leaf leaf-b"></div>
                <div class="leaf leaf-c"></div>

                <div class="drop drop-a"></div>
                <div class="drop drop-b"></div>
                <div class="drop drop-c"></div>

                <div class="serum-bottle">
                    <div class="serum-label">
                        DERMAFORM<br>
                        <small>BOTANICAL SERUM</small>
                    </div>
                </div>

                <div class="cream-jar">
                    <div class="cream-label">
                        HYDRATING<br>CREAM
                    </div>
                </div>
            </div>
        </div>
        """
    )

    st.markdown("")

    c1, c2, c3 = st.columns(3)

    with c1:
        if st.button(
            "🧪 Create Formula",
            use_container_width=True,
        ):
            st.session_state.page = "🧴 Formulation Lab"
            st.rerun()

    with c2:
        if st.button(
            "🌿 Explore Ingredients",
            use_container_width=True,
        ):
            st.session_state.page = "🌿 Ingredient Library"
            st.rerun()

    with c3:
        if st.button(
            "⚗️ Chemistry Tools",
            use_container_width=True,
        ):
            st.session_state.page = "⚗️ Chemistry Tools"
            st.rerun()

    st.markdown("")

    st.html(
        """
        <div class="soft-card">
            <div class="feature-title">Explore the Lab</div>
            <div class="feature-text">
                Everything you need to explore cosmetic formulation chemistry
                in one skincare-inspired educational workspace.
            </div>
        </div>
        """
    )

    st.markdown("")

    features = [
        (
            "🧴",
            "Formulation Lab",
            "Build formulas with ingredients, percentages, phases and batch sizes.",
        ),
        (
            "🌿",
            "Ingredient Library",
            "Explore INCI names, functions, solubility, use ranges and pH considerations.",
        ),
        (
            "⚗️",
            "Chemistry Tools",
            "Perform concentration, dilution, batch-size and pH calculations.",
        ),
        (
            "🧬",
            "Compatibility",
            "Screen ingredient pairs using transparent educational rules.",
        ),
        (
            "📈",
            "Stability",
            "Review educational indicators for pH, phases, sensitivity and preservation.",
        ),
        (
            "📄",
            "Formula Report",
            "Turn your current formulation into a clean academic report.",
        ),
    ]

    first_row = st.columns(3)

    for column, feature in zip(first_row, features[:3]):
        with column:
            icon, title, text = feature
            st.html(
                f"""
                <div class="feature-card">
                    <div class="feature-icon">{icon}</div>
                    <div class="feature-title">{title}</div>
                    <div class="feature-text">{text}</div>
                </div>
                """
            )

    st.markdown("")

    second_row = st.columns(3)

    for column, feature in zip(second_row, features[3:]):
        with column:
            icon, title, text = feature
            st.html(
                f"""
                <div class="feature-card">
                    <div class="feature-icon">{icon}</div>
                    <div class="feature-title">{title}</div>
                    <div class="feature-text">{text}</div>
                </div>
                """
            )


# ============================================================
# DASHBOARD
# ============================================================

elif st.session_state.page == "📊 Dashboard":
    st.html(
        """
        <div class="page-title">Dashboard</div>
        <div class="page-subtitle">
            Your skincare formulation workspace at a glance.
        </div>
        """
    )

    metrics = [
        (
            "🧴",
            str(len(st.session_state.formula)),
            "Ingredients in Formula",
        ),
        (
            "🌿",
            str(len(INGREDIENTS)),
            "Ingredients Available",
        ),
        (
            "💧",
            f"{formula_total():.1f}%",
            "Formula Loaded",
        ),
        (
            "📦",
            f"{st.session_state.batch_size:.0f} g",
            "Current Batch",
        ),
        (
            "📈",
            f"{stability_score()}%",
            "Stability Estimate",
        ),
    ]

    metric_columns = st.columns(5)

    for column, metric in zip(metric_columns, metrics):
        with column:
            icon, value, label = metric
            st.html(
                f"""
                <div class="metric-card">
                    <div class="metric-icon">{icon}</div>
                    <div class="metric-value">{value}</div>
                    <div class="metric-label">{label}</div>
                </div>
                """
            )

    st.markdown("")

    left, right = st.columns([1.45, 1])

    with left:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">🧴 Current Formula</div>
                <div class="feature-text">
                    Your active formulation and ingredient distribution.
                </div>
            </div>
            """
        )

        if st.session_state.formula:
            st.dataframe(
                formula_dataframe(),
                use_container_width=True,
                hide_index=True,
            )
        else:
            st.info(
                "No formula loaded yet. Open the Formulation Lab to begin."
            )

    with right:
        distribution = ingredient_category_distribution()

        if distribution:
            chart_df = pd.DataFrame(
                {
                    "Category": list(distribution.keys()),
                    "Percentage": list(distribution.values()),
                }
            )

            fig = px.pie(
                chart_df,
                names="Category",
                values="Percentage",
                hole=0.48,
                title="Formula Categories",
            )

            fig.update_layout(
                paper_bgcolor="rgba(0,0,0,0)",
                plot_bgcolor="rgba(0,0,0,0)",
                font=dict(
                    family="Arial",
                    color="#403631",
                ),
            )

            st.plotly_chart(
                fig,
                use_container_width=True,
            )
        else:
            st.html(
                """
                <div class="soft-card">
                    <div class="feature-title">📊 Formula analytics</div>
                    <div class="feature-text">
                        Add ingredients to generate distribution charts.
                    </div>
                </div>
                """
            )

    st.markdown("")

    if formula_total() > 100:
        st.error(
            f"Formula is over 100% by {formula_total() - 100:.2f}%."
        )
    elif formula_total() < 100 and st.session_state.formula:
        st.warning(
            f"Formula currently totals {formula_total():.2f}%. "
            f"{remaining_percentage():.2f}% remains."
        )
    elif formula_total() == 100:
        st.success("Formula totals exactly 100%.")


# ============================================================
# FORMULATION LAB
# ============================================================

elif st.session_state.page == "🧴 Formulation Lab":
    st.html(
        """
        <div class="page-title">🧴 Formulation Lab</div>
        <div class="page-subtitle">
            Design a skincare formulation in a digital product-development workspace.
        </div>
        """
    )

    setup_a, setup_b, setup_c = st.columns(3)

    with setup_a:
        st.session_state.formula_name = st.text_input(
            "Formula Name",
            value=st.session_state.formula_name,
        )

    with setup_b:
        product_options = [
            "Serum",
            "Moisturizer",
            "Cream",
            "Cleanser",
            "Toner",
            "Gel",
        ]

        st.session_state.product_type = st.selectbox(
            "Product Type",
            product_options,
            index=product_options.index(
                st.session_state.product_type
            ),
        )

    with setup_c:
        st.session_state.batch_size = st.number_input(
            "Batch Size (g)",
            min_value=1.0,
            max_value=10000.0,
            value=float(st.session_state.batch_size),
            step=10.0,
        )

    st.markdown("")

    preset_col, preset_button_col = st.columns([3, 1])

    with preset_col:
        preset_name = st.selectbox(
            "Quick-start formulation",
            list(FORMULA_PRESETS.keys()),
        )

    with preset_button_col:
        st.markdown("<br>", unsafe_allow_html=True)
        if st.button(
            "✨ Load Preset",
            use_container_width=True,
        ):
            load_preset(preset_name)
            st.rerun()

    st.markdown("")

    left, middle, right = st.columns(
        [1.0, 1.55, 0.85]
    )

    # --------------------------------------------------------
    # ADD INGREDIENT
    # --------------------------------------------------------

    with left:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">🌿 Add Ingredient</div>
                <div class="feature-text">
                    Select a cosmetic ingredient and assign its percentage and phase.
                </div>
            </div>
            """
        )

        selected_ingredient = st.selectbox(
            "Ingredient",
            list(INGREDIENTS.keys()),
        )

        ingredient_data = INGREDIENTS[selected_ingredient]

        st.caption(
            f'{ingredient_data["icon"]} '
            f'{ingredient_data["function"]}'
        )

        percentage = st.number_input(
            "Percentage (%)",
            min_value=0.0,
            max_value=100.0,
            value=1.0,
            step=0.1,
        )

        phase_options = [
            "Water Phase",
            "Oil Phase",
            "Surfactant Phase",
            "Cool Down Phase",
        ]

        selected_phase = st.selectbox(
            "Formulation Phase",
            phase_options,
        )

        if st.button(
            "＋ Add to Formula",
            use_container_width=True,
        ):
            success, message = add_ingredient(
                selected_ingredient,
                percentage,
                selected_phase,
            )

            if success:
                st.success(message)
                st.rerun()
            else:
                st.error(message)

        if st.button(
            "🗑 Clear Formula",
            use_container_width=True,
        ):
            st.session_state.formula = []
            st.rerun()

    # --------------------------------------------------------
    # CURRENT FORMULA
    # --------------------------------------------------------

    with middle:
        st.html(
            f"""
            <div class="soft-card">
                <div class="feature-title">
                    🧴 {st.session_state.formula_name}
                </div>
                <div class="feature-text">
                    {st.session_state.product_type}
                    &nbsp;•&nbsp;
                    {st.session_state.batch_size:.0f} g batch
                </div>
            </div>
            """
        )

        st.markdown("")

        if not st.session_state.formula:
            st.info(
                "Your formula is empty. Add an ingredient or load a preset."
            )

        for index, item in enumerate(
            st.session_state.formula
        ):
            data = INGREDIENTS[item["name"]]
            grams = calculate_quantity(
                item["percentage"],
                st.session_state.batch_size,
            )

            st.html(
                f"""
                <div class="ingredient-card">
                    <div class="ingredient-name">
                        {data["icon"]} {item["name"]}
                    </div>
                    <div class="ingredient-meta">
                        {item["function"]} • {item["phase"]}<br>
                        INCI: {item["inci"]}
                    </div>
                </div>
                """
            )

            row_a, row_b, row_c = st.columns(
                [1, 1, 0.7]
            )

            with row_a:
                st.write(
                    f"**{item['percentage']:.2f}%**"
                )

            with row_b:
                st.write(
                    f"**{grams:.2f} g**"
                )

            with row_c:
                if st.button(
                    "Remove",
                    key=f"remove_formula_{index}",
                ):
                    st.session_state.formula.pop(index)
                    st.rerun()

        total = formula_total()

        if st.session_state.formula:
            st.markdown("")

            progress_value = int(
                max(0, min(100, total))
            )

            st.progress(progress_value)

            if total < 100:
                st.caption(
                    f"Formula: {total:.2f}% • "
                    f"Remaining: {100 - total:.2f}%"
                )
            elif total == 100:
                st.success(
                    "Formula is balanced at 100%."
                )
            else:
                st.error(
                    f"Formula exceeds 100% by {total - 100:.2f}%."
                )

    # --------------------------------------------------------
    # PRODUCT PREVIEW
    # --------------------------------------------------------

    with right:
        st.html(
            f"""
            <div class="soft-card" style="text-align:center;">
                <div class="feature-title">Product Preview</div>

                <div style="
                    font-size:82px;
                    margin:25px 0 16px;
                ">
                    🧴
                </div>

                <div style="
                    font-family:Georgia,serif;
                    font-size:20px;
                    color:#332B27;
                ">
                    {st.session_state.product_type}
                </div>

                <div style="
                    color:#625750;
                    font-size:11px;
                    margin-top:6px;
                ">
                    {st.session_state.batch_size:.0f} g prototype
                </div>
            </div>
            """
        )

        st.markdown("")

        if total <= 100:
            st.success(
                f"✓ {total:.1f}% allocated"
            )
        else:
            st.error(
                "⚠ Formula exceeds 100%"
            )

    # --------------------------------------------------------
    # FORMULA TABLE
    # --------------------------------------------------------

    st.markdown("")

    st.html(
        """
        <div class="soft-card">
            <div class="feature-title">📋 Formula Composition</div>
            <div class="feature-text">
                Ingredient-level composition with calculated batch quantities.
            </div>
        </div>
        """
    )

    if st.session_state.formula:
        df = formula_dataframe()

        st.dataframe(
            df,
            use_container_width=True,
            hide_index=True,
        )

        st.download_button(
            "📥 Download Formula CSV",
            df.to_csv(index=False),
            file_name="DERMAFORM_formula.csv",
            mime="text/csv",
        )


# ============================================================
# INGREDIENT LIBRARY
# ============================================================

elif st.session_state.page == "🌿 Ingredient Library":
    st.html(
        """
        <div class="page-title">🌿 Ingredient Library</div>
        <div class="page-subtitle">
            Explore the chemistry and formulation role of skincare ingredients.
        </div>
        """
    )

    search = st.text_input(
        "🔍 Search ingredients",
        placeholder="Try glycerin, aloe, niacinamide or tocopherol",
    )

    categories = [
        "All",
        "Hydrators",
        "Botanicals",
        "Actives",
        "Emollients",
        "Surfactants",
        "pH Adjusters",
        "Thickeners",
        "Base",
    ]

    category = st.selectbox(
        "Filter by category",
        categories,
    )

    filtered = []

    for name, data in INGREDIENTS.items():
        search_text = " ".join(
            [
                name,
                data["inci"],
                data["function"],
                data["role"],
                data["category"],
            ]
        ).lower()

        matches_search = (
            not search
            or search.lower() in search_text
        )

        matches_category = (
            category == "All"
            or data["category"] == category
        )

        if matches_search and matches_category:
            filtered.append((name, data))

    st.caption(
        f"{len(filtered)} ingredient(s) found"
    )

    for start in range(
        0,
        len(filtered),
        2,
    ):
        columns = st.columns(2)

        for column, item in zip(
            columns,
            filtered[start:start + 2],
        ):
            name, data = item

            with column:
                st.html(
                    f"""
                    <div class="feature-card" style="margin-bottom:15px;">
                        <div class="feature-icon">
                            {data["icon"]}
                        </div>

                        <div class="feature-title">
                            {name}
                        </div>

                        <div class="feature-text">
                            <b>INCI:</b> {data["inci"]}<br>
                            <b>Function:</b> {data["function"]}<br>
                            <b>Skincare role:</b> {data["role"]}<br>
                            <b>Phase:</b> {data["phase"]}<br>
                            <b>Solubility:</b> {data["solubility"]}<br>
                            <b>Typical use:</b> {data["use"]}<br>
                            <b>pH considerations:</b> {data["ph"]}
                        </div>
                    </div>
                    """
                )


# ============================================================
# CHEMISTRY TOOLS
# ============================================================

elif st.session_state.page == "⚗️ Chemistry Tools":
    st.html(
        """
        <div class="page-title">⚗️ Chemistry Tools</div>
        <div class="page-subtitle">
            Interactive calculations supporting cosmetic formulation learning.
        </div>
        """
    )

    tabs = st.tabs(
        [
            "💧 Concentration",
            "🧪 Dilution",
            "📦 Batch Size",
            "⚗️ pH Explorer",
        ]
    )

    # --------------------------------------------------------
    # CONCENTRATION
    # --------------------------------------------------------

    with tabs[0]:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">
                    💧 Concentration Calculator
                </div>
                <div class="feature-text">
                    Calculate ingredient mass from concentration and batch size.
                    <br><br>
                    <b>Mass = Percentage × Batch Size ÷ 100</b>
                </div>
            </div>
            """
        )

        col_a, col_b = st.columns(2)

        with col_a:
            concentration = st.number_input(
                "Concentration (%)",
                min_value=0.0,
                max_value=100.0,
                value=5.0,
                step=0.1,
            )

        with col_b:
            batch = st.number_input(
                "Batch Size (g)",
                min_value=0.1,
                value=100.0,
                step=10.0,
            )

        result = calculate_quantity(
            concentration,
            batch,
        )

        st.metric(
            "Required ingredient mass",
            f"{result:.2f} g",
        )

    # --------------------------------------------------------
    # DILUTION
    # --------------------------------------------------------

    with tabs[1]:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">
                    🧪 Dilution Calculator
                </div>
                <div class="feature-text">
                    Use the educational dilution relationship:
                    <br><br>
                    <b>C₁V₁ = C₂V₂</b>
                </div>
            </div>
            """
        )

        col_a, col_b = st.columns(2)

        with col_a:
            c1 = st.number_input(
                "C₁ — Stock concentration",
                min_value=0.001,
                value=10.0,
                step=0.5,
            )

            c2 = st.number_input(
                "C₂ — Desired concentration",
                min_value=0.001,
                value=2.0,
                step=0.5,
            )

        with col_b:
            v2 = st.number_input(
                "V₂ — Final volume",
                min_value=0.01,
                value=100.0,
                step=10.0,
            )

        if c2 <= c1:
            v1 = (c2 * v2) / c1
            diluent = max(0, v2 - v1)

            st.metric(
                "V₁ — Stock solution required",
                f"{v1:.2f}",
            )

            st.info(
                f"Approximate diluent amount: {diluent:.2f}"
            )
        else:
            st.error(
                "For a simple dilution, the desired concentration "
                "should not exceed the stock concentration."
            )

    # --------------------------------------------------------
    # BATCH SIZE
    # --------------------------------------------------------

    with tabs[2]:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">
                    📦 Batch Size Calculator
                </div>
                <div class="feature-text">
                    Calculate the ingredient quantity required for a selected batch.
                </div>
            </div>
            """
        )

        batch_size = st.number_input(
            "Target batch size (g)",
            min_value=1.0,
            value=100.0,
            step=10.0,
        )

        ingredient_percentage = st.number_input(
            "Ingredient percentage (%)",
            min_value=0.0,
            max_value=100.0,
            value=5.0,
            step=0.1,
        )

        quantity = calculate_quantity(
            ingredient_percentage,
            batch_size,
        )

        st.metric(
            "Ingredient quantity",
            f"{quantity:.2f} g",
        )

    # --------------------------------------------------------
    # PH
    # --------------------------------------------------------

    with tabs[3]:
        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">
                    ⚗️ pH Explorer
                </div>
                <div class="feature-text">
                    Explore the educational relationship:
                    <br><br>
                    <b>pH = −log₁₀[H⁺]</b>
                </div>
            </div>
            """
        )

        hydrogen = st.number_input(
            "[H⁺] concentration (mol/L)",
            min_value=1e-12,
            max_value=1.0,
            value=1e-6,
            format="%.10f",
        )

        calculated_ph = -np.log10(hydrogen)

        st.metric(
            "Calculated pH",
            f"{calculated_ph:.2f}",
        )

        if calculated_ph < 7:
            st.info(
                "The calculated solution is acidic."
            )
        elif calculated_ph == 7:
            st.success(
                "The calculated solution is neutral."
            )
        else:
            st.warning(
                "The calculated solution is basic/alkaline."
            )


# ============================================================
# COMPATIBILITY PAGE
# ============================================================

elif st.session_state.page == "🧬 Compatibility":
    st.html(
        """
        <div class="page-title">🧬 Compatibility Checker</div>
        <div class="page-subtitle">
            Educational ingredient interaction screening for formulation planning.
        </div>
        """
    )

    left, right = st.columns(2)

    with left:
        ingredient_a = st.selectbox(
            "Ingredient A",
            list(INGREDIENTS.keys()),
        )

    with right:
        ingredient_b = st.selectbox(
            "Ingredient B",
            list(INGREDIENTS.keys()),
            index=1,
        )

    result, status_color, explanation = compatibility_pair(
        ingredient_a,
        ingredient_b,
    )

    st.markdown("")

    if status_color == "green":
        css_class = "status status-green"
        icon = "🟢"
    elif status_color == "yellow":
        css_class = "status status-yellow"
        icon = "🟡"
    else:
        css_class = "status status-red"
        icon = "🔴"

    st.html(
        f"""
        <div class="{css_class}">
            {icon} {result}
        </div>
        """
    )

    st.markdown("")

    st.html(
        f"""
        <div class="soft-card">
            <div class="feature-title">
                🔬 Chemistry Interpretation
            </div>
            <div class="feature-text">
                {explanation}
            </div>
        </div>
        """
    )

    st.markdown("")

    detail_left, detail_right = st.columns(2)

    for column, name in zip(
        [detail_left, detail_right],
        [ingredient_a, ingredient_b],
    ):
        data = INGREDIENTS[name]

        with column:
            st.html(
                f"""
                <div class="feature-card">
                    <div class="feature-icon">
                        {data["icon"]}
                    </div>

                    <div class="feature-title">
                        {name}
                    </div>

                    <div class="feature-text">
                        <b>INCI:</b> {data["inci"]}<br>
                        <b>Function:</b> {data["function"]}<br>
                        <b>Role:</b> {data["role"]}<br>
                        <b>Phase:</b> {data["phase"]}<br>
                        <b>Solubility:</b> {data["solubility"]}<br>
                        <b>Typical use:</b> {data["use"]}<br>
                        <b>pH:</b> {data["ph"]}
                    </div>
                </div>
                """
            )

    st.markdown("")

    st.html(
        """
        <div class="disclaimer">
            ⚠️ <b>Educational limitation:</b>
            This checker is a simplified academic screening tool.
            Real formulation compatibility depends on concentration,
            pH, raw-material grade, processing conditions, packaging,
            preservation system and laboratory testing.
        </div>
        """
    )


# ============================================================
# STABILITY PAGE
# ============================================================

elif st.session_state.page == "📈 Stability":
    st.html(
        """
        <div class="page-title">📈 Stability Analysis</div>
        <div class="page-subtitle">
            Skincare-product quality indicators based on the current educational formula.
        </div>
        """
    )

    score = stability_score()

    metric_a, metric_b, metric_c = st.columns(3)

    with metric_a:
        st.html(
            f"""
            <div class="metric-card">
                <div class="metric-icon">📈</div>
                <div class="metric-value">{score}%</div>
                <div class="metric-label">Educational Stability Estimate</div>
            </div>
            """
        )

    with metric_b:
        st.html(
            """
            <div class="metric-card">
                <div class="metric-icon">⚗️</div>
                <div class="metric-value">
                    Review
                </div>
                <div class="metric-label">pH Considerations</div>
            </div>
            """
        )

    with metric_c:
        st.html(
            """
            <div class="metric-card">
                <div class="metric-icon">🧴</div>
                <div class="metric-value">
                    Monitor
                </div>
                <div class="metric-label">Phase Compatibility</div>
            </div>
            """
        )

    st.markdown("")

    factors = {
        "pH Considerations": 78,
        "Ingredient Sensitivity": 82,
        "Phase Compatibility": 86,
        "Preservation Considerations": 75,
    }

    st.html(
        """
        <div class="soft-card">
            <div class="feature-title">
                🧪 Stability Factors
            </div>
            <div class="feature-text">
                These indicators are for educational demonstration only.
            </div>
        </div>
        """
    )

    for factor, value in factors.items():
        st.write(
            f"**{factor}** — {value}%"
        )
        st.progress(value)

    chart_df = pd.DataFrame(
        {
            "Factor": list(factors.keys()),
            "Score": list(factors.values()),
        }
    )

    fig = px.bar(
        chart_df,
        x="Factor",
        y="Score",
        range_y=[0, 100],
        title="Educational Stability Indicators",
    )

    fig.update_layout(
        paper_bgcolor="rgba(0,0,0,0)",
        plot_bgcolor="rgba(0,0,0,0)",
        font=dict(
            family="Arial",
            color="#403631",
        ),
    )

    st.plotly_chart(
        fig,
        use_container_width=True,
    )

    st.html(
        """
        <div class="disclaimer">
            ⚠️ <b>Important:</b>
            The stability estimate is an educational model, not a laboratory
            result. Actual cosmetic stability requires appropriate physical,
            chemical and microbiological testing under controlled conditions.
        </div>
        """
    )


# ============================================================
# FORMULA REPORT
# ============================================================

elif st.session_state.page == "📄 Formula Report":
    st.html(
        """
        <div class="page-title">📄 Formula Report</div>
        <div class="page-subtitle">
            A clean formulation summary for your college demonstration.
        </div>
        """
    )

    st.html(
        f"""
        <div class="report-header">
            <div class="report-product">
                🧴 {st.session_state.formula_name}
            </div>

            <div class="report-meta">
                <b>Product type:</b>
                {st.session_state.product_type}
                &nbsp;&nbsp;•&nbsp;&nbsp;

                <b>Batch size:</b>
                {st.session_state.batch_size:.0f} g
            </div>
        </div>
        """
    )

    st.markdown("")

    if st.session_state.formula:
        report_df = formula_dataframe()

        st.html(
            """
            <div class="soft-card">
                <div class="feature-title">
                    🧪 Formula Composition
                </div>
                <div class="feature-text">
                    Ingredient, INCI, function, phase and calculated quantity.
                </div>
            </div>
            """
        )

        st.dataframe(
            report_df,
            use_container_width=True,
            hide_index=True,
        )

        st.markdown("")

        left, right = st.columns(2)

        with left:
            st.html(
                """
                <div class="soft-card">
                    <div class="feature-title">
                        📊 Ingredient Distribution
                    </div>
                </div>
                """
            )

            pie_fig = px.pie(
                report_df,
                names="Ingredient",
                values="Percentage (%)",
                hole=0.45,
            )

            pie_fig.update_layout(
                paper_bgcolor="rgba(0,0,0,0)",
                font=dict(
                    family="Arial",
                    color="#403631",
                ),
            )

            st.plotly_chart(
                pie_fig,
                use_container_width=True,
            )

        with right:
            phase_data = phase_distribution()

            if phase_data:
                phase_df = pd.DataFrame(
                    {
                        "Phase": list(phase_data.keys()),
                        "Percentage": list(
                            phase_data.values()
                        ),
                    }
                )

                phase_fig = px.bar(
                    phase_df,
                    x="Phase",
                    y="Percentage",
                    title="Phase Distribution",
                )

                phase_fig.update_layout(
                    paper_bgcolor="rgba(0,0,0,0)",
                    plot_bgcolor="rgba(0,0,0,0)",
                    font=dict(
                        family="Arial",
                        color="#403631",
                    ),
                )

                st.plotly_chart(
                    phase_fig,
                    use_container_width=True,
                )

    else:
        st.info(
            "No ingredients have been added. "
            "Open Formulation Lab and create a formula first."
        )

    st.markdown("")

    st.html(
        """
        <div class="soft-card">
            <div class="feature-title">
                🧬 Compatibility
            </div>
            <div class="feature-text">
                Current formula can be reviewed using the Compatibility page.
            </div>
        </div>
        """
    )

    st.success(
        "🟢 Educational compatibility screening available."
    )

    st.markdown("")

    report_score = stability_score()

    st.html(
        """
        <div class="soft-card">
            <div class="feature-title">
                📈 Stability Estimate
            </div>
            <div class="feature-text">
                Educational indicator based on formula structure.
            </div>
        </div>
        """
    )

    st.progress(report_score)

    st.write(
        f"Estimated educational score: **{report_score}%**"
    )

    st.markdown("")

    st.html(
        """
        <div class="disclaimer">
            🎓 <b>Academic-use notice:</b>
            DERMAFORM is a college educational project.
            Its calculations and screening models are intended to demonstrate
            cosmetic chemistry concepts and do not certify a product as safe,
            effective, stable or suitable for commercial manufacture.
        </div>
        """
    )

    if st.session_state.formula:
        st.markdown("")

        st.download_button(
            "📥 Export Formula Report as CSV",
            formula_dataframe().to_csv(index=False),
            file_name="DERMAFORM_formula_report.csv",
            mime="text/csv",
        )


# ============================================================
# FOOTER
# ============================================================

st.html(
    """
    <div class="footer">
        🧪 <b>DERMAFORM</b> • Smart Skincare Formulation Lab<br>
        Application of Chemistry in Cosmetic &amp; Skincare Formulation<br>
        🎓 Educational College Project
    </div>
    """
)


# ============================================================
# END OF APPLICATION
# ============================================================
# Notes:
# 1. This application is intentionally skincare-focused.
# 2. Chemistry is represented through formulation calculations.
# 3. Raw HTML is rendered through st.html(), not st.markdown().
# 4. This prevents the HTML-code boxes visible in the old version.
# 5. Text colors were darkened for better readability.
# 6. External font loading is not required.
# 7. Georgia is used for premium editorial-style headings.
# 8. Arial is used for readable interface text.
# 9. The application remains compatible with Streamlit 1.63.0.
# 10. No image assets are required.
# 11. No separate CSS file is required.
# 12. No JavaScript is required.
# 13. All calculations are performed locally.
# 14. Formula quantities update with batch size.
# 15. Formula percentages are checked against 100%.
# 16. Preset formulas can be loaded instantly.
# 17. Ingredient cards can be searched and filtered.
# 18. Compatibility has an explicit educational limitation.
# 19. Stability has an explicit educational limitation.
# 20. Actual cosmetic testing must be performed in a laboratory.
# ============================================================
