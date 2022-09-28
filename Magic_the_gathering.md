{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 50,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "        <script type=\"text/javascript\">\n",
       "        window.PlotlyConfig = {MathJaxConfig: 'local'};\n",
       "        if (window.MathJax) {MathJax.Hub.Config({SVG: {font: \"STIX-Web\"}});}\n",
       "        if (typeof require !== 'undefined') {\n",
       "        require.undef(\"plotly\");\n",
       "        requirejs.config({\n",
       "            paths: {\n",
       "                'plotly': ['https://cdn.plot.ly/plotly-latest.min']\n",
       "            }\n",
       "        });\n",
       "        require(['plotly'], function(Plotly) {\n",
       "            window._Plotly = Plotly;\n",
       "        });\n",
       "        }\n",
       "        </script>\n",
       "        "
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "import numpy as np # linear algebra\n",
    "import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)\n",
    "import matplotlib as mpl\n",
    "import matplotlib.pyplot as plt\n",
    "import seaborn as sns\n",
    "import warnings; warnings.filterwarnings(action='once')\n",
    "import re\n",
    "import json\n",
    "\n",
    "# plotly\n",
    "# import plotly.plotly as py\n",
    "from plotly.offline import init_notebook_mode, iplot, plot\n",
    "import plotly as py\n",
    "init_notebook_mode(connected=True)\n",
    "import plotly.graph_objs as go"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Перед вами набор данных, который содержит все когда-либо выпущенные карты Magic: The Gathering. Для большего понимания, о чем идет речь, вы можете перейти на официальный сайт игры и ознакомиться с базовыми правилами.\n",
    "\n",
    " \n",
    "На основе этих данных составьте отчет, который будет содержать следующую информацию:\n",
    "\n",
    " \n",
    "1) Распределение карт по цвету в зависимости от редкости.\n",
    "\n",
    "2) Процент карт, запрещенных в формате Commander, а также распределение по типу для этих карт.\n",
    "\n",
    "3) Топ-10 карт, не являющихся землями, которые были напечатаны в наибольшем количестве сетов.\n",
    "\n",
    "4) Для карт, не являющихся землями, определите, какая часть из них даёт ману с помощью своего эффекта. Покажите распределение по типу маны, который дают эти карты. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "Skipping line 66241: unexpected end of data\n"
     ]
    }
   ],
   "source": [
    "df = pd.read_csv('/mnt/HC_Volume_18315164/home-jupyter/jupyter-j-postnova/Magic/all_mtg_cards.csv', error_bad_lines=False, engine=\"python\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>name</th>\n",
       "      <th>multiverse_id</th>\n",
       "      <th>layout</th>\n",
       "      <th>names</th>\n",
       "      <th>mana_cost</th>\n",
       "      <th>cmc</th>\n",
       "      <th>colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>type</th>\n",
       "      <th>supertypes</th>\n",
       "      <th>...</th>\n",
       "      <th>foreign_names</th>\n",
       "      <th>printings</th>\n",
       "      <th>original_text</th>\n",
       "      <th>original_type</th>\n",
       "      <th>legalities</th>\n",
       "      <th>source</th>\n",
       "      <th>image_url</th>\n",
       "      <th>set</th>\n",
       "      <th>set_name</th>\n",
       "      <th>id</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>130550.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{5}{W}{W}</td>\n",
       "      <td>7.0</td>\n",
       "      <td>['White']</td>\n",
       "      <td>['W']</td>\n",
       "      <td>Creature — Human Cleric</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>[{'name': 'Ausgewählter der Ahnfrau', 'text': ...</td>\n",
       "      <td>['10E', 'JUD', 'UMA']</td>\n",
       "      <td>First strike (This creature deals combat damag...</td>\n",
       "      <td>Creature - Human Cleric</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>5f8287b1-5bb6-5f4c-ad17-316a40d5bb0c</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>NaN</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{5}{W}{W}</td>\n",
       "      <td>7.0</td>\n",
       "      <td>['White']</td>\n",
       "      <td>['W']</td>\n",
       "      <td>Creature — Human Cleric</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>['10E', 'JUD', 'UMA']</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>b7c19924-b4bf-56fc-aa73-f586e940bd42</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>2 rows × 39 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "                name  multiverse_id  layout  names  mana_cost  cmc     colors  \\\n",
       "0  Ancestor's Chosen       130550.0  normal    NaN  {5}{W}{W}  7.0  ['White']   \n",
       "1  Ancestor's Chosen            NaN  normal    NaN  {5}{W}{W}  7.0  ['White']   \n",
       "\n",
       "  color_identity                     type supertypes  ...  \\\n",
       "0          ['W']  Creature — Human Cleric        NaN  ...   \n",
       "1          ['W']  Creature — Human Cleric        NaN  ...   \n",
       "\n",
       "                                       foreign_names              printings  \\\n",
       "0  [{'name': 'Ausgewählter der Ahnfrau', 'text': ...  ['10E', 'JUD', 'UMA']   \n",
       "1                                                NaN  ['10E', 'JUD', 'UMA']   \n",
       "\n",
       "                                       original_text            original_type  \\\n",
       "0  First strike (This creature deals combat damag...  Creature - Human Cleric   \n",
       "1                                                NaN                      NaN   \n",
       "\n",
       "                                          legalities source  \\\n",
       "0  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "1  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "\n",
       "                                           image_url  set       set_name  \\\n",
       "0  http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "1                                                NaN  10E  Tenth Edition   \n",
       "\n",
       "                                     id  \n",
       "0  5f8287b1-5bb6-5f4c-ad17-316a40d5bb0c  \n",
       "1  b7c19924-b4bf-56fc-aa73-f586e940bd42  \n",
       "\n",
       "[2 rows x 39 columns]"
      ]
     },
     "execution_count": 3,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head(2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\n",
      "RangeIndex: 66239 entries, 0 to 66238\n",
      "Data columns (total 39 columns):\n",
      " #   Column          Non-Null Count  Dtype  \n",
      "---  ------          --------------  -----  \n",
      " 0   name            66239 non-null  object \n",
      " 1   multiverse_id   46961 non-null  float64\n",
      " 2   layout          66239 non-null  object \n",
      " 3   names           0 non-null      float64\n",
      " 4   mana_cost       57713 non-null  object \n",
      " 5   cmc             66239 non-null  float64\n",
      " 6   colors          52258 non-null  object \n",
      " 7   color_identity  59267 non-null  object \n",
      " 8   type            66239 non-null  object \n",
      " 9   supertypes      9839 non-null   object \n",
      " 10  subtypes        40625 non-null  object \n",
      " 11  rarity          66239 non-null  object \n",
      " 12  text            65277 non-null  object \n",
      " 13  flavor          34154 non-null  object \n",
      " 14  artist          66228 non-null  object \n",
      " 15  number          66239 non-null  object \n",
      " 16  power           31103 non-null  object \n",
      " 17  toughness       31103 non-null  object \n",
      " 18  loyalty         1068 non-null   object \n",
      " 19  variations      12326 non-null  object \n",
      " 20  watermark       5105 non-null   object \n",
      " 21  border          0 non-null      float64\n",
      " 22  timeshifted     0 non-null      float64\n",
      " 23  hand            119 non-null    float64\n",
      " 24  life            119 non-null    float64\n",
      " 25  reserved        0 non-null      float64\n",
      " 26  release_date    0 non-null      float64\n",
      " 27  starter         0 non-null      float64\n",
      " 28  rulings         36826 non-null  object \n",
      " 29  foreign_names   40803 non-null  object \n",
      " 30  printings       66239 non-null  object \n",
      " 31  original_text   46014 non-null  object \n",
      " 32  original_type   46941 non-null  object \n",
      " 33  legalities      64849 non-null  object \n",
      " 34  source          0 non-null      float64\n",
      " 35  image_url       46961 non-null  object \n",
      " 36  set             66239 non-null  object \n",
      " 37  set_name        66239 non-null  object \n",
      " 38  id              66239 non-null  object \n",
      "dtypes: float64(11), object(28)\n",
      "memory usage: 19.7+ MB\n"
     ]
    }
   ],
   "source": [
    "#посмотрим информацию о данных\n",
    "df.info()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [],
   "source": [
    "#информация о цвете карт находится в двух столбцах: colors и color_identity, название карты в столбце name\n",
    "#из данныхпо таблице сразу видно, что имен гораздо больше, чем цветов, также у многих карт нет значения colors, \n",
    "#но есть значение color_identity\n",
    "#имена карт повторяются"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "23960"
      ]
     },
     "execution_count": 6,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#имена карт повторяются, посомтрим, сколько уникальных имен\n",
    "df.name.nunique()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {},
   "outputs": [],
   "source": [
    "#хорошо, тогда если взять уникальные имена, сколько значений будет иметь информацию о colors или color_identity?"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "20582"
      ]
     },
     "execution_count": 8,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_colors = df.query(\"colors.notnull()\")\n",
    "df_colors.name.nunique()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "21481"
      ]
     },
     "execution_count": 9,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_color_identity = df.query(\"color_identity.notnull()\")\n",
    "df_color_identity.name.nunique()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [],
   "source": [
    "#отлично, тогда возьмем лучшее из имеющегося. Объединим df_color_identity and df_colors и уберем дубликаты по имени"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_merged = df_colors.merge(df_color_identity, how = 'outer').drop_duplicates(subset='name')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {},
   "outputs": [],
   "source": [
    "#таблица df_merged содержит столько же строк, сколько и df_color_identity, это значит, \n",
    "#что у каждого уникального имени точно есть color_identity - с ним и будем работать"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_name_color = df_merged[['name','color_identity']]\n",
    "# оставить только нужные столбцы "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {
    "scrolled": false
   },
   "outputs": [],
   "source": [
    "df_name_color = df_name_color.reset_index(drop=True) #сбросить индексы строк, чтобы итеррироваться по ним"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {},
   "outputs": [],
   "source": [
    "# used to convert a string column to a list of the strings\n",
    "def str_to_list(cell):\n",
    "    cell = ''.join(c for c in cell if c not in \"'[]\")\n",
    "    cell = cell.split(', ')\n",
    "    return cell"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {},
   "outputs": [],
   "source": [
    "for i in range(len(df_name_color.color_identity)):\n",
    "    df_name_color.color_identity[i] = str_to_list(df_name_color.color_identity[i])"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# 1) Распределение карт по цвету в зависимости от редкости."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "'U'"
      ]
     },
     "execution_count": 17,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_name_color.color_identity[21476][3]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {
    "scrolled": false
   },
   "outputs": [],
   "source": [
    "df_name_color.insert(2, \"W\", 0)\n",
    "df_name_color.insert(3, \"B\", 0)\n",
    "df_name_color.insert(4, \"U\", 0)\n",
    "df_name_color.insert(5, \"R\", 0)\n",
    "df_name_color.insert(6, \"G\", 0)\n",
    "# добавимнеобходимые столбцы вдатафрейм"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0"
      ]
     },
     "execution_count": 19,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_name_color['W'][1]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/opt/tljh/user/lib/python3.7/site-packages/ipykernel_launcher.py:3: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n"
     ]
    }
   ],
   "source": [
    "for i in range(len(df_name_color.color_identity)):\n",
    "    if 'W' in df_name_color.color_identity[i]:\n",
    "        df_name_color.W[i] = 1\n",
    "    if 'B' in df_name_color.color_identity[i]:\n",
    "        df_name_color.B[i] = 1\n",
    "    if 'U' in df_name_color.color_identity[i]:\n",
    "        df_name_color.U[i] = 1\n",
    "    if 'R' in df_name_color.color_identity[i]:\n",
    "        df_name_color.R[i] = 1\n",
    "    if 'G' in df_name_color.color_identity[i]:\n",
    "        df_name_color.G[i] = 1"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>name</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>W</th>\n",
       "      <th>B</th>\n",
       "      <th>U</th>\n",
       "      <th>R</th>\n",
       "      <th>G</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>Angel of Mercy</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>Angelic Blessing</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>Angelic Chorus</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>Angelic Wall</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21476</th>\n",
       "      <td>Jack-in-the-Mox</td>\n",
       "      <td>[B, G, R, U, W]</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21477</th>\n",
       "      <td>Serra's Sanctum</td>\n",
       "      <td>[W]</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21478</th>\n",
       "      <td>Sap Sucker</td>\n",
       "      <td>[G]</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21479</th>\n",
       "      <td>Magosi, the Waterveil</td>\n",
       "      <td>[U]</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21480</th>\n",
       "      <td>Piranha Marsh</td>\n",
       "      <td>[B]</td>\n",
       "      <td>0</td>\n",
       "      <td>1</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>21481 rows × 7 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "                        name   color_identity  W  B  U  R  G\n",
       "0          Ancestor's Chosen              [W]  1  0  0  0  0\n",
       "1             Angel of Mercy              [W]  1  0  0  0  0\n",
       "2           Angelic Blessing              [W]  1  0  0  0  0\n",
       "3             Angelic Chorus              [W]  1  0  0  0  0\n",
       "4               Angelic Wall              [W]  1  0  0  0  0\n",
       "...                      ...              ... .. .. .. .. ..\n",
       "21476        Jack-in-the-Mox  [B, G, R, U, W]  1  1  1  1  1\n",
       "21477        Serra's Sanctum              [W]  1  0  0  0  0\n",
       "21478             Sap Sucker              [G]  0  0  0  0  1\n",
       "21479  Magosi, the Waterveil              [U]  0  0  1  0  0\n",
       "21480          Piranha Marsh              [B]  0  1  0  0  0\n",
       "\n",
       "[21481 rows x 7 columns]"
      ]
     },
     "execution_count": 21,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_name_color"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.axes._subplots.AxesSubplot at 0x7fba680eda90>"
      ]
     },
     "execution_count": 22,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "df_name_color_sum = pd.DataFrame(df_name_color[['W','B','U','R','G']].sum())\n",
    "df_name_color_sum.plot.bar()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 23,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.axes._subplots.AxesSubplot at 0x7fba680ba1d0>"
      ]
     },
     "execution_count": 23,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "df_name_color_sum.plot()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 24,
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "W_dif = 100 - df_name_color_sum.loc['W']/df_name_color_sum.max()*100\n",
    "B_dif = 100 - df_name_color_sum.loc['B']/df_name_color_sum.max()*100\n",
    "U_dif = 100 - df_name_color_sum.loc['U']/df_name_color_sum.max()*100\n",
    "R_dif = 100 - df_name_color_sum.loc['R']/df_name_color_sum.max()*100\n",
    "G_dif = 100 - df_name_color_sum.loc['G']/df_name_color_sum.max()*100"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(0    1.027984\n",
       " dtype: float64,\n",
       " 0    0.0\n",
       " dtype: float64,\n",
       " 0    1.846564\n",
       " dtype: float64,\n",
       " 0    1.351609\n",
       " dtype: float64,\n",
       " 0    2.779364\n",
       " dtype: float64)"
      ]
     },
     "execution_count": 25,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "W_dif, B_dif, U_dif, R_dif, G_dif #разница между макисмальным значением и остальными впроцентах"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# На основе полученных данных можно сказать, что пристуствие каждого цвета в картах распределно равномерно, максимальная разница в количестве между двумя цветами 2,77%. \n",
    "\n",
    "Это, вероятно не новость, скорее всего так и задумано игрой, что все цвета должны быть примерно в одинаковом количестве. Тем не менее, такой анализ может быть полезен например в случае выпуска новой колекции карт, для того, чтобы уровнять количество задействованных цветов, или для того, чтобы убедится, что разбивка по цветам происходит правильно."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Также интересно посомтреть, как распределяются __комбинации__ этих цветов в картах"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 26,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Для этого вернемсяк датафрейму df_merged"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 27,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_color_combo = df_merged[['name','color_identity']] \n",
    "# возьмем нужные столбцы"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>name</th>\n",
       "      <th>color_identity</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>Angel of Mercy</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>Angelic Blessing</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>Angelic Chorus</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>Angelic Wall</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>58703</th>\n",
       "      <td>Jack-in-the-Mox</td>\n",
       "      <td>['B', 'G', 'R', 'U', 'W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>58766</th>\n",
       "      <td>Serra's Sanctum</td>\n",
       "      <td>['W']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>58791</th>\n",
       "      <td>Sap Sucker</td>\n",
       "      <td>['G']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>59219</th>\n",
       "      <td>Magosi, the Waterveil</td>\n",
       "      <td>['U']</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>59221</th>\n",
       "      <td>Piranha Marsh</td>\n",
       "      <td>['B']</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>21481 rows × 2 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "                        name             color_identity\n",
       "0          Ancestor's Chosen                      ['W']\n",
       "2             Angel of Mercy                      ['W']\n",
       "4           Angelic Blessing                      ['W']\n",
       "6             Angelic Chorus                      ['W']\n",
       "7               Angelic Wall                      ['W']\n",
       "...                      ...                        ...\n",
       "58703        Jack-in-the-Mox  ['B', 'G', 'R', 'U', 'W']\n",
       "58766        Serra's Sanctum                      ['W']\n",
       "58791             Sap Sucker                      ['G']\n",
       "59219  Magosi, the Waterveil                      ['U']\n",
       "59221          Piranha Marsh                      ['B']\n",
       "\n",
       "[21481 rows x 2 columns]"
      ]
     },
     "execution_count": 28,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_color_combo"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 29,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>color_identity</th>\n",
       "      <th>amount</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>['B', 'G', 'R', 'U', 'W']</td>\n",
       "      <td>81</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>['B', 'G', 'R', 'U']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>['B', 'G', 'R', 'W']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>['B', 'G', 'R']</td>\n",
       "      <td>73</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>['B', 'G', 'U', 'W']</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>['B', 'G', 'U']</td>\n",
       "      <td>40</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>['B', 'G', 'W']</td>\n",
       "      <td>32</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>['B', 'G']</td>\n",
       "      <td>276</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>['B', 'R', 'U', 'W']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>['B', 'R', 'U']</td>\n",
       "      <td>79</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>['B', 'R', 'W']</td>\n",
       "      <td>43</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>['B', 'R']</td>\n",
       "      <td>334</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>['B', 'U', 'W']</td>\n",
       "      <td>67</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>['B', 'U']</td>\n",
       "      <td>333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>14</th>\n",
       "      <td>['B', 'W']</td>\n",
       "      <td>265</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>15</th>\n",
       "      <td>['B']</td>\n",
       "      <td>3621</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>16</th>\n",
       "      <td>['G', 'R', 'U', 'W']</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>17</th>\n",
       "      <td>['G', 'R', 'U']</td>\n",
       "      <td>32</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>18</th>\n",
       "      <td>['G', 'R', 'W']</td>\n",
       "      <td>76</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>19</th>\n",
       "      <td>['G', 'R']</td>\n",
       "      <td>323</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>20</th>\n",
       "      <td>['G', 'U', 'W']</td>\n",
       "      <td>68</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21</th>\n",
       "      <td>['G', 'U']</td>\n",
       "      <td>253</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>22</th>\n",
       "      <td>['G', 'W']</td>\n",
       "      <td>324</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>23</th>\n",
       "      <td>['G']</td>\n",
       "      <td>3519</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>24</th>\n",
       "      <td>['R', 'U', 'W']</td>\n",
       "      <td>39</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25</th>\n",
       "      <td>['R', 'U']</td>\n",
       "      <td>254</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>26</th>\n",
       "      <td>['R', 'W']</td>\n",
       "      <td>264</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>27</th>\n",
       "      <td>['R']</td>\n",
       "      <td>3575</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>28</th>\n",
       "      <td>['U', 'W']</td>\n",
       "      <td>332</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>29</th>\n",
       "      <td>['U']</td>\n",
       "      <td>3568</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>30</th>\n",
       "      <td>['W']</td>\n",
       "      <td>3598</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "               color_identity  amount\n",
       "0   ['B', 'G', 'R', 'U', 'W']      81\n",
       "1        ['B', 'G', 'R', 'U']       2\n",
       "2        ['B', 'G', 'R', 'W']       2\n",
       "3             ['B', 'G', 'R']      73\n",
       "4        ['B', 'G', 'U', 'W']       3\n",
       "5             ['B', 'G', 'U']      40\n",
       "6             ['B', 'G', 'W']      32\n",
       "7                  ['B', 'G']     276\n",
       "8        ['B', 'R', 'U', 'W']       2\n",
       "9             ['B', 'R', 'U']      79\n",
       "10            ['B', 'R', 'W']      43\n",
       "11                 ['B', 'R']     334\n",
       "12            ['B', 'U', 'W']      67\n",
       "13                 ['B', 'U']     333\n",
       "14                 ['B', 'W']     265\n",
       "15                      ['B']    3621\n",
       "16       ['G', 'R', 'U', 'W']       3\n",
       "17            ['G', 'R', 'U']      32\n",
       "18            ['G', 'R', 'W']      76\n",
       "19                 ['G', 'R']     323\n",
       "20            ['G', 'U', 'W']      68\n",
       "21                 ['G', 'U']     253\n",
       "22                 ['G', 'W']     324\n",
       "23                      ['G']    3519\n",
       "24            ['R', 'U', 'W']      39\n",
       "25                 ['R', 'U']     254\n",
       "26                 ['R', 'W']     264\n",
       "27                      ['R']    3575\n",
       "28                 ['U', 'W']     332\n",
       "29                      ['U']    3568\n",
       "30                      ['W']    3598"
      ]
     },
     "execution_count": 29,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# сгруппируем по color_identity и посчитаем количество\n",
    "df_color_combo_amount = df_color_combo.groupby('color_identity')\\\n",
    "    .count()\\\n",
    "    .reset_index()\\\n",
    "    .rename(columns = {'name':'amount'})\\\n",
    "    .sort_values('color_identity', ascending=True)\n",
    "\n",
    "df_color_combo_amount"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/opt/tljh/user/lib/python3.7/site-packages/ipykernel_launcher.py:6: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n"
     ]
    }
   ],
   "source": [
    "# посчитаем количество элементов в списке\n",
    "# NB!!! поскольку на данный моент в столбце находится строка, и для того, чтобы посчитать количество\n",
    "# элементов нужно распарсить ее в список, проще посчитать количество разделителейи прибавить еще 1\n",
    "df_color_combo_amount['sum_colors'] = 0 \n",
    "for i in range(len(df_color_combo_amount.color_identity)):\n",
    "    df_color_combo_amount['sum_colors'][i] =  df_color_combo_amount.color_identity[i].count(',') + 1"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_color_combo_amount = df_color_combo_amount.set_index('sum_colors')\n",
    "# устанвить количество элементов списке цветов индексом"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 32,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_color_combo_amount = df_color_combo_amount.sort_values('sum_colors', ascending=True)\n",
    "# сортировка по индексу"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 33,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.axes._subplots.AxesSubplot at 0x7fba6802aef0>"
      ]
     },
     "execution_count": 33,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "df_color_combo_amount.plot.bar()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Из барплота видно, что самая большая доля карт приходится на карты с 1 цветом, затем количество карт уменьшается с увеличением количества цветов, но интересно, что карт с количеством цветов 5 больше, чем карт с количеством цветов 4."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Также можно посомтреть, какую долю занимают комбинации с разным количество цветов в игре"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>sum_colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>amount</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1</td>\n",
       "      <td>['B']</td>\n",
       "      <td>3621</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1</td>\n",
       "      <td>['R']</td>\n",
       "      <td>3575</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1</td>\n",
       "      <td>['G']</td>\n",
       "      <td>3519</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1</td>\n",
       "      <td>['U']</td>\n",
       "      <td>3568</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1</td>\n",
       "      <td>['W']</td>\n",
       "      <td>3598</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>2</td>\n",
       "      <td>['U', 'W']</td>\n",
       "      <td>332</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>2</td>\n",
       "      <td>['B', 'G']</td>\n",
       "      <td>276</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>2</td>\n",
       "      <td>['R', 'W']</td>\n",
       "      <td>264</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>2</td>\n",
       "      <td>['R', 'U']</td>\n",
       "      <td>254</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>2</td>\n",
       "      <td>['B', 'R']</td>\n",
       "      <td>334</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>2</td>\n",
       "      <td>['G', 'W']</td>\n",
       "      <td>324</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>2</td>\n",
       "      <td>['B', 'U']</td>\n",
       "      <td>333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>2</td>\n",
       "      <td>['B', 'W']</td>\n",
       "      <td>265</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>2</td>\n",
       "      <td>['G', 'U']</td>\n",
       "      <td>253</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>14</th>\n",
       "      <td>2</td>\n",
       "      <td>['G', 'R']</td>\n",
       "      <td>323</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>15</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'G', 'U']</td>\n",
       "      <td>40</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>16</th>\n",
       "      <td>3</td>\n",
       "      <td>['R', 'U', 'W']</td>\n",
       "      <td>39</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>17</th>\n",
       "      <td>3</td>\n",
       "      <td>['G', 'U', 'W']</td>\n",
       "      <td>68</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>18</th>\n",
       "      <td>3</td>\n",
       "      <td>['G', 'R', 'U']</td>\n",
       "      <td>32</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>19</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'G', 'R']</td>\n",
       "      <td>73</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>20</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'U', 'W']</td>\n",
       "      <td>67</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'R', 'W']</td>\n",
       "      <td>43</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>22</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'R', 'U']</td>\n",
       "      <td>79</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>23</th>\n",
       "      <td>3</td>\n",
       "      <td>['B', 'G', 'W']</td>\n",
       "      <td>32</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>24</th>\n",
       "      <td>3</td>\n",
       "      <td>['G', 'R', 'W']</td>\n",
       "      <td>76</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25</th>\n",
       "      <td>4</td>\n",
       "      <td>['B', 'G', 'U', 'W']</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>26</th>\n",
       "      <td>4</td>\n",
       "      <td>['G', 'R', 'U', 'W']</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>27</th>\n",
       "      <td>4</td>\n",
       "      <td>['B', 'G', 'R', 'W']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>28</th>\n",
       "      <td>4</td>\n",
       "      <td>['B', 'R', 'U', 'W']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>29</th>\n",
       "      <td>4</td>\n",
       "      <td>['B', 'G', 'R', 'U']</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>30</th>\n",
       "      <td>5</td>\n",
       "      <td>['B', 'G', 'R', 'U', 'W']</td>\n",
       "      <td>81</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "    sum_colors             color_identity  amount\n",
       "0            1                      ['B']    3621\n",
       "1            1                      ['R']    3575\n",
       "2            1                      ['G']    3519\n",
       "3            1                      ['U']    3568\n",
       "4            1                      ['W']    3598\n",
       "5            2                 ['U', 'W']     332\n",
       "6            2                 ['B', 'G']     276\n",
       "7            2                 ['R', 'W']     264\n",
       "8            2                 ['R', 'U']     254\n",
       "9            2                 ['B', 'R']     334\n",
       "10           2                 ['G', 'W']     324\n",
       "11           2                 ['B', 'U']     333\n",
       "12           2                 ['B', 'W']     265\n",
       "13           2                 ['G', 'U']     253\n",
       "14           2                 ['G', 'R']     323\n",
       "15           3            ['B', 'G', 'U']      40\n",
       "16           3            ['R', 'U', 'W']      39\n",
       "17           3            ['G', 'U', 'W']      68\n",
       "18           3            ['G', 'R', 'U']      32\n",
       "19           3            ['B', 'G', 'R']      73\n",
       "20           3            ['B', 'U', 'W']      67\n",
       "21           3            ['B', 'R', 'W']      43\n",
       "22           3            ['B', 'R', 'U']      79\n",
       "23           3            ['B', 'G', 'W']      32\n",
       "24           3            ['G', 'R', 'W']      76\n",
       "25           4       ['B', 'G', 'U', 'W']       3\n",
       "26           4       ['G', 'R', 'U', 'W']       3\n",
       "27           4       ['B', 'G', 'R', 'W']       2\n",
       "28           4       ['B', 'R', 'U', 'W']       2\n",
       "29           4       ['B', 'G', 'R', 'U']       2\n",
       "30           5  ['B', 'G', 'R', 'U', 'W']      81"
      ]
     },
     "execution_count": 34,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_2 = df_color_combo_amount.reset_index()\n",
    "df_2"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 35,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>sum_colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>amount</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>2</td>\n",
       "      <td>10</td>\n",
       "      <td>10</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>3</td>\n",
       "      <td>10</td>\n",
       "      <td>10</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>4</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>5</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   sum_colors  color_identity  amount\n",
       "0           1               5       5\n",
       "1           2              10      10\n",
       "2           3              10      10\n",
       "3           4               5       5\n",
       "4           5               1       1"
      ]
     },
     "execution_count": 35,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_pie = df_2.groupby('sum_colors', as_index=False).count()\n",
    "df_pie"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "31"
      ]
     },
     "execution_count": 36,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_pie.color_identity.sum()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 37,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_pie['part'] = df_pie['amount'] / 31 *100"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 38,
   "metadata": {},
   "outputs": [],
   "source": [
    "# df_pie.set_index('color_identity')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>sum_colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>amount</th>\n",
       "      <th>part</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>16.129032</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>2</td>\n",
       "      <td>10</td>\n",
       "      <td>10</td>\n",
       "      <td>32.258065</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>3</td>\n",
       "      <td>10</td>\n",
       "      <td>10</td>\n",
       "      <td>32.258065</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>4</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>16.129032</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>5</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>3.225806</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   sum_colors  color_identity  amount       part\n",
       "0           1               5       5  16.129032\n",
       "1           2              10      10  32.258065\n",
       "2           3              10      10  32.258065\n",
       "3           4               5       5  16.129032\n",
       "4           5               1       1   3.225806"
      ]
     },
     "execution_count": 39,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_pie"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 40,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "x = df_pie.amount\n",
    "labels = df_pie.sum_colors\n",
    "\n",
    "fig, ax = plt.subplots()\n",
    "ax.pie(x, labels=labels)\n",
    "ax.set_title('Colors combos')\n",
    "plt.tight_layout()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# 2) Процент карт, запрещенных в формате Commander, а также распределение по типу для этих карт."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 41,
   "metadata": {},
   "outputs": [],
   "source": [
    "# данные по форматам хранятся в столбце legalities\n",
    "# посмотрим на них"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\n",
      "RangeIndex: 66239 entries, 0 to 66238\n",
      "Data columns (total 39 columns):\n",
      " #   Column          Non-Null Count  Dtype  \n",
      "---  ------          --------------  -----  \n",
      " 0   name            66239 non-null  object \n",
      " 1   multiverse_id   46961 non-null  float64\n",
      " 2   layout          66239 non-null  object \n",
      " 3   names           0 non-null      float64\n",
      " 4   mana_cost       57713 non-null  object \n",
      " 5   cmc             66239 non-null  float64\n",
      " 6   colors          52258 non-null  object \n",
      " 7   color_identity  59267 non-null  object \n",
      " 8   type            66239 non-null  object \n",
      " 9   supertypes      9839 non-null   object \n",
      " 10  subtypes        40625 non-null  object \n",
      " 11  rarity          66239 non-null  object \n",
      " 12  text            65277 non-null  object \n",
      " 13  flavor          34154 non-null  object \n",
      " 14  artist          66228 non-null  object \n",
      " 15  number          66239 non-null  object \n",
      " 16  power           31103 non-null  object \n",
      " 17  toughness       31103 non-null  object \n",
      " 18  loyalty         1068 non-null   object \n",
      " 19  variations      12326 non-null  object \n",
      " 20  watermark       5105 non-null   object \n",
      " 21  border          0 non-null      float64\n",
      " 22  timeshifted     0 non-null      float64\n",
      " 23  hand            119 non-null    float64\n",
      " 24  life            119 non-null    float64\n",
      " 25  reserved        0 non-null      float64\n",
      " 26  release_date    0 non-null      float64\n",
      " 27  starter         0 non-null      float64\n",
      " 28  rulings         36826 non-null  object \n",
      " 29  foreign_names   40803 non-null  object \n",
      " 30  printings       66239 non-null  object \n",
      " 31  original_text   46014 non-null  object \n",
      " 32  original_type   46941 non-null  object \n",
      " 33  legalities      64849 non-null  object \n",
      " 34  source          0 non-null      float64\n",
      " 35  image_url       46961 non-null  object \n",
      " 36  set             66239 non-null  object \n",
      " 37  set_name        66239 non-null  object \n",
      " 38  id              66239 non-null  object \n",
      "dtypes: float64(11), object(28)\n",
      "memory usage: 19.7+ MB\n"
     ]
    }
   ],
   "source": [
    "df.info()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "metadata": {},
   "outputs": [],
   "source": [
    "# В данных есть нулевые строки, уберем их"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>index</th>\n",
       "      <th>name</th>\n",
       "      <th>multiverse_id</th>\n",
       "      <th>layout</th>\n",
       "      <th>names</th>\n",
       "      <th>mana_cost</th>\n",
       "      <th>cmc</th>\n",
       "      <th>colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>type</th>\n",
       "      <th>...</th>\n",
       "      <th>foreign_names</th>\n",
       "      <th>printings</th>\n",
       "      <th>original_text</th>\n",
       "      <th>original_type</th>\n",
       "      <th>legalities</th>\n",
       "      <th>source</th>\n",
       "      <th>image_url</th>\n",
       "      <th>set</th>\n",
       "      <th>set_name</th>\n",
       "      <th>id</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0</td>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>130550.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{5}{W}{W}</td>\n",
       "      <td>7.0</td>\n",
       "      <td>['White']</td>\n",
       "      <td>['W']</td>\n",
       "      <td>Creature — Human Cleric</td>\n",
       "      <td>...</td>\n",
       "      <td>[{'name': 'Ausgewählter der Ahnfrau', 'text': ...</td>\n",
       "      <td>['10E', 'JUD', 'UMA']</td>\n",
       "      <td>First strike (This creature deals combat damag...</td>\n",
       "      <td>Creature - Human Cleric</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>5f8287b1-5bb6-5f4c-ad17-316a40d5bb0c</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1</td>\n",
       "      <td>Ancestor's Chosen</td>\n",
       "      <td>NaN</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{5}{W}{W}</td>\n",
       "      <td>7.0</td>\n",
       "      <td>['White']</td>\n",
       "      <td>['W']</td>\n",
       "      <td>Creature — Human Cleric</td>\n",
       "      <td>...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>['10E', 'JUD', 'UMA']</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>b7c19924-b4bf-56fc-aa73-f586e940bd42</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>2 rows × 40 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "   index               name  multiverse_id  layout  names  mana_cost  cmc  \\\n",
       "0      0  Ancestor's Chosen       130550.0  normal    NaN  {5}{W}{W}  7.0   \n",
       "1      1  Ancestor's Chosen            NaN  normal    NaN  {5}{W}{W}  7.0   \n",
       "\n",
       "      colors color_identity                     type  ...  \\\n",
       "0  ['White']          ['W']  Creature — Human Cleric  ...   \n",
       "1  ['White']          ['W']  Creature — Human Cleric  ...   \n",
       "\n",
       "                                       foreign_names              printings  \\\n",
       "0  [{'name': 'Ausgewählter der Ahnfrau', 'text': ...  ['10E', 'JUD', 'UMA']   \n",
       "1                                                NaN  ['10E', 'JUD', 'UMA']   \n",
       "\n",
       "                                       original_text            original_type  \\\n",
       "0  First strike (This creature deals combat damag...  Creature - Human Cleric   \n",
       "1                                                NaN                      NaN   \n",
       "\n",
       "                                          legalities source  \\\n",
       "0  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "1  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "\n",
       "                                           image_url  set       set_name  \\\n",
       "0  http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "1                                                NaN  10E  Tenth Edition   \n",
       "\n",
       "                                     id  \n",
       "0  5f8287b1-5bb6-5f4c-ad17-316a40d5bb0c  \n",
       "1  b7c19924-b4bf-56fc-aa73-f586e940bd42  \n",
       "\n",
       "[2 rows x 40 columns]"
      ]
     },
     "execution_count": 44,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df2 = df.query(\"legalities.notnull()\").reset_index() #сбросить индекс, чтобы итерироваться по фрейму\n",
    "df2.head(2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 45,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "\"[{'format': 'Commander', 'legality': 'Legal'}, {'format': 'Duel', 'legality': 'Legal'}, {'format': 'Legacy', 'legality': 'Legal'}, {'format': 'Modern', 'legality': 'Legal'}, {'format': 'Paupercommander', 'legality': 'Restricted'}, {'format': 'Penny', 'legality': 'Legal'}, {'format': 'Premodern', 'legality': 'Legal'}, {'format': 'Vintage', 'legality': 'Legal'}]\""
      ]
     },
     "execution_count": 45,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df2.legalities[1] #даные хранятся в виде строки, нужно вытащить информацию о 'format': 'Commander'\n",
    "# не разрешенные значения записаны как Banned"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Создадим из строковых значений словари"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 46,
   "metadata": {},
   "outputs": [],
   "source": [
    "def str_to_list_2(cell):\n",
    "    cell = ''.join(c for c in cell if c not in \"'[]\")\n",
    "    cell = cell.split(' / ')\n",
    "    return cell"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 47,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/opt/tljh/user/lib/python3.7/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n"
     ]
    }
   ],
   "source": [
    "for i in range(len(df2)):   \n",
    "    df2.legalities[i] = str_to_list_2(df2.legalities[i].replace(\"'\", '\"').replace('}, {', '} / {'))\n",
    "    \n",
    "#подготовка данных, заменить разделитель, для того, что бы создать список"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 48,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_test2 = df2 \n",
    "# пересохранить датафрейм, чтобы тестировать варианты"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 51,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/opt/tljh/user/lib/python3.7/site-packages/ipykernel_launcher.py:10: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n",
      "/opt/tljh/user/lib/python3.7/site-packages/pandas/core/indexing.py:670: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n"
     ]
    }
   ],
   "source": [
    "df_test2['Commander'] = 0\n",
    "for i in range(len(df_test2.legalities)):\n",
    "    #initialising the string \n",
    "    for m in range(len(df_test2.legalities[i])):\n",
    "        if 'Commander' in df_test2.legalities[i][m]:\n",
    "            #initialising the string \n",
    "            string_1 = df_test2.legalities[i][m]\n",
    "            #using json.loads() \n",
    "            res_dict=json.loads(string_1) \n",
    "            df_test2['Commander'][i] = res_dict.get('legality')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 52,
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "df_final_commander = df_test2.groupby('Commander', as_index=False).agg('name').count()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 53,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "1    351\n",
       "Name: name, dtype: int64"
      ]
     },
     "execution_count": 53,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "banned = df_final_commander.query(\"Commander == 'Banned'\").name\n",
    "banned"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Таким образом, количество карт, запрещенных в формате Commander 351"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 54,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "2    64225\n",
       "Name: name, dtype: int64"
      ]
     },
     "execution_count": 54,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "legal = df_final_commander.query(\"Commander == 'Legal'\").name\n",
    "legal"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 55,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Commander</th>\n",
       "      <th>name</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0</td>\n",
       "      <td>273</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>Banned</td>\n",
       "      <td>351</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>Legal</td>\n",
       "      <td>64225</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "  Commander   name\n",
       "0         0    273\n",
       "1    Banned    351\n",
       "2     Legal  64225"
      ]
     },
     "execution_count": 55,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_final_commander"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 56,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0.55"
      ]
     },
     "execution_count": 56,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "round (351 / 64225 *100, 2)\n",
    "# расчет веду именно среди тех карт, где legalities указано, а также, где в принципе есть формат Commander"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Процент карт, запрещенных в формате Commander  0.55%"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 57,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>type</th>\n",
       "      <th>name</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>Artifact</td>\n",
       "      <td>81</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>Artifact Creature — Golem</td>\n",
       "      <td>6</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>Conspiracy</td>\n",
       "      <td>26</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>Creature — Avatar</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>Creature — Devil</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>Creature — Efreet</td>\n",
       "      <td>4</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>Creature — Giant</td>\n",
       "      <td>7</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>Creature — Horror</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>Creature — Human Nomad</td>\n",
       "      <td>6</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>Creature — Human Wizard</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>Creature — Merfolk Pirate</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>Enchantment</td>\n",
       "      <td>36</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>Enchantment — Aura</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>Instant</td>\n",
       "      <td>19</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>14</th>\n",
       "      <td>Land</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>15</th>\n",
       "      <td>Legendary Artifact</td>\n",
       "      <td>4</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>16</th>\n",
       "      <td>Legendary Artifact Creature — Scout</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>17</th>\n",
       "      <td>Legendary Creature — Angel</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>18</th>\n",
       "      <td>Legendary Creature — Demon</td>\n",
       "      <td>7</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>19</th>\n",
       "      <td>Legendary Creature — Eldrazi</td>\n",
       "      <td>9</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>20</th>\n",
       "      <td>Legendary Creature — Elemental Otter</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21</th>\n",
       "      <td>Legendary Creature — Elf Advisor</td>\n",
       "      <td>4</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>22</th>\n",
       "      <td>Legendary Creature — Elf Druid</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>23</th>\n",
       "      <td>Legendary Creature — Human Minion</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>24</th>\n",
       "      <td>Legendary Creature — Moonfolk Monk</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25</th>\n",
       "      <td>Legendary Enchantment</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>26</th>\n",
       "      <td>Legendary Land</td>\n",
       "      <td>12</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>27</th>\n",
       "      <td>Sorcery</td>\n",
       "      <td>96</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                                    type  name\n",
       "0                               Artifact    81\n",
       "1              Artifact Creature — Golem     6\n",
       "2                             Conspiracy    26\n",
       "3                      Creature — Avatar     1\n",
       "4                       Creature — Devil     2\n",
       "5                      Creature — Efreet     4\n",
       "6                       Creature — Giant     7\n",
       "7                      Creature — Horror     1\n",
       "8                 Creature — Human Nomad     6\n",
       "9                Creature — Human Wizard     3\n",
       "10             Creature — Merfolk Pirate     3\n",
       "11                           Enchantment    36\n",
       "12                    Enchantment — Aura     1\n",
       "13                               Instant    19\n",
       "14                                  Land     5\n",
       "15                    Legendary Artifact     4\n",
       "16   Legendary Artifact Creature — Scout     3\n",
       "17            Legendary Creature — Angel     3\n",
       "18            Legendary Creature — Demon     7\n",
       "19          Legendary Creature — Eldrazi     9\n",
       "20  Legendary Creature — Elemental Otter     5\n",
       "21      Legendary Creature — Elf Advisor     4\n",
       "22        Legendary Creature — Elf Druid     2\n",
       "23     Legendary Creature — Human Minion     3\n",
       "24    Legendary Creature — Moonfolk Monk     1\n",
       "25                 Legendary Enchantment     1\n",
       "26                        Legendary Land    12\n",
       "27                               Sorcery    96"
      ]
     },
     "execution_count": 57,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_type_disp = df_test2.query(\"Commander == 'Banned'\").groupby('type', as_index=False).agg('name').count()\n",
    "df_type_disp"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 58,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.axes._subplots.AxesSubplot at 0x7fba6811aef0>"
      ]
     },
     "execution_count": 58,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "df_type_disp.plot.bar()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 59,
   "metadata": {},
   "outputs": [],
   "source": [
    "#распределение карт по типу"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Топ-10 карт, не являющихся землями, которые были напечатаны в наибольшем количестве сетов."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 60,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_3 = df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 61,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>set_name</th>\n",
       "      <th>amount_of_sets</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>344</th>\n",
       "      <td>Magic Online Promos</td>\n",
       "      <td>2219</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>396</th>\n",
       "      <td>Mystery Booster</td>\n",
       "      <td>1662</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>460</th>\n",
       "      <td>Salvat 2005</td>\n",
       "      <td>717</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>503</th>\n",
       "      <td>The List</td>\n",
       "      <td>711</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>94</th>\n",
       "      <td>Commander Legends</td>\n",
       "      <td>667</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>257</th>\n",
       "      <td>Innistrad: Double Feature</td>\n",
       "      <td>618</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>310</th>\n",
       "      <td>Kamigawa: Neon Dynasty</td>\n",
       "      <td>523</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>475</th>\n",
       "      <td>Secret Lair Drop</td>\n",
       "      <td>517</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>500</th>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>487</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>297</th>\n",
       "      <td>Jumpstart</td>\n",
       "      <td>486</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                      set_name  amount_of_sets\n",
       "344        Magic Online Promos            2219\n",
       "396            Mystery Booster            1662\n",
       "460                Salvat 2005             717\n",
       "503                   The List             711\n",
       "94           Commander Legends             667\n",
       "257  Innistrad: Double Feature             618\n",
       "310     Kamigawa: Neon Dynasty             523\n",
       "475           Secret Lair Drop             517\n",
       "500              Tenth Edition             487\n",
       "297                  Jumpstart             486"
      ]
     },
     "execution_count": 61,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# выбрать значения с типом - земля, сгрупировать и посчитать по сетам, отсортировать, взять топ 10\n",
    "df_3.query(\"type != 'Land' and type != 'Legendary Land'\").groupby('set_name', as_index=False).agg('name')\\\n",
    "    .count().rename(columns = {'name' : 'amount_of_sets', })\\\n",
    "    .sort_values('amount_of_sets', ascending = False)\\\n",
    "    .head(10)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# 4) Для карт, не являющихся землями, определите, какая часть из них даёт ману с помощью своего эффекта. Покажите распределение по типу маны, который дают эти карты."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Информация о том, что карта дает ману содержится в тексте карты, для начала поищем по ключевому слову, чтобы посмотреть, в каких конструкциях содержитсянеобходимая информация"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 133,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_mana = df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 134,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "\"If an opponent had three or more cards put into their graveyard from anywhere this turn, you may pay {0} rather than pay this spell's mana cost.\\nExile all cards from target player's graveyard.\""
      ]
     },
     "execution_count": 134,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_mana['mana_is_in'] = df_mana['original_text'].str.lower().str.contains('mana', flags=re.IGNORECASE, regex=True)\n",
    "df_mana_contains = pd.DataFrame(df_mana.query(\"mana_is_in == True\").original_text).reset_index()\n",
    "df_mana_contains.original_text[110]"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Add {W}{U}{B}{R}{G} to your mana pool.'\n",
    "Double the amount of each type of mana in your mana pool.'\n",
    "Add {1} to your mana pool.\n",
    "Add 1 green mana to your mana pool.\n",
    "Add 2 colorless mana to your mana pool.\n",
    "Adds 3 mana of any single color of your choice to your mana pool,\n",
    "Add 2 colorless mana to your mana pool.\n",
    "\n",
    "В основном данные о мане содержаться между словами Add и mana"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 135,
   "metadata": {},
   "outputs": [],
   "source": [
    "#напишем функцию, которая ищет вхождение и возвращет занчение между ними\n",
    "def parse(text):\n",
    "    dict = {}\n",
    "    m = re.search('dd (.+?) mana', text)\n",
    "    if m:\n",
    "        value = m.group(1)\n",
    "        dict['Add_mana'] = value\n",
    "    else: dict['Add_mana'] = 0\n",
    "    return dict "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 136,
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "#ограничить датафрей не землями\n",
    "df_mana = df.query(\"type != 'Land' and type != 'Legendary Land' and original_text.notnull()\")\\\n",
    "    .drop_duplicates('name')\\\n",
    "    .reset_index()\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 137,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_mana_no_lands = df_mana.query(\"mana_is_in == True\").reset_index()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 138,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>level_0</th>\n",
       "      <th>index</th>\n",
       "      <th>name</th>\n",
       "      <th>multiverse_id</th>\n",
       "      <th>layout</th>\n",
       "      <th>names</th>\n",
       "      <th>mana_cost</th>\n",
       "      <th>cmc</th>\n",
       "      <th>colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>...</th>\n",
       "      <th>printings</th>\n",
       "      <th>original_text</th>\n",
       "      <th>original_type</th>\n",
       "      <th>legalities</th>\n",
       "      <th>source</th>\n",
       "      <th>image_url</th>\n",
       "      <th>set</th>\n",
       "      <th>set_name</th>\n",
       "      <th>id</th>\n",
       "      <th>mana_is_in</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>88</td>\n",
       "      <td>128</td>\n",
       "      <td>March of the Machines</td>\n",
       "      <td>106555.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{3}{U}</td>\n",
       "      <td>4.0</td>\n",
       "      <td>['Blue']</td>\n",
       "      <td>['U']</td>\n",
       "      <td>...</td>\n",
       "      <td>['10E', 'ARC', 'MRD']</td>\n",
       "      <td>Each noncreature artifact is an artifact creat...</td>\n",
       "      <td>Enchantment</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>60961bae-4655-5a6b-843d-f6180c2c310d</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>128</td>\n",
       "      <td>184</td>\n",
       "      <td>Consume Spirit</td>\n",
       "      <td>129505.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{X}{1}{B}</td>\n",
       "      <td>2.0</td>\n",
       "      <td>['Black']</td>\n",
       "      <td>['B']</td>\n",
       "      <td>...</td>\n",
       "      <td>['10E', '9ED', 'DDC', 'DPA', 'DVD', 'HOP', 'M1...</td>\n",
       "      <td>Spend only black mana on X.\\nConsume Spirit de...</td>\n",
       "      <td>Sorcery</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>26180f1b-41a1-5fa5-9f6b-39ee08a1a203</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>209</td>\n",
       "      <td>294</td>\n",
       "      <td>Manabarbs</td>\n",
       "      <td>130367.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{3}{R}</td>\n",
       "      <td>4.0</td>\n",
       "      <td>['Red']</td>\n",
       "      <td>['R']</td>\n",
       "      <td>...</td>\n",
       "      <td>['10E', '2ED', '3ED', '4BB', '4ED', '5ED', '6E...</td>\n",
       "      <td>Whenever a player taps a land for mana, Manaba...</td>\n",
       "      <td>Enchantment</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>85245362-de1a-59fd-9f9a-d6a0c9522592</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>243</td>\n",
       "      <td>340</td>\n",
       "      <td>Birds of Paradise</td>\n",
       "      <td>129906.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{G}</td>\n",
       "      <td>1.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>['10E', '2ED', '3ED', '4BB', '4ED', '5ED', '6E...</td>\n",
       "      <td>Flying (This creature can't be blocked except ...</td>\n",
       "      <td>Creature - Bird</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>28e56ac7-6a09-5118-9ade-6755b051bc0b</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>259</td>\n",
       "      <td>364</td>\n",
       "      <td>Joiner Adept</td>\n",
       "      <td>130500.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{1}{G}</td>\n",
       "      <td>2.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>['10E', '5DN']</td>\n",
       "      <td>Lands you control have \"{T}: Add one mana of a...</td>\n",
       "      <td>Creature - Elf Druid</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>2f39f144-a02a-5017-a45a-4a3cfc2c7a36</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2324</th>\n",
       "      <td>22512</td>\n",
       "      <td>66019</td>\n",
       "      <td>Runeflare Trap</td>\n",
       "      <td>197536.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{4}{R}{R}</td>\n",
       "      <td>6.0</td>\n",
       "      <td>['Red']</td>\n",
       "      <td>['R']</td>\n",
       "      <td>...</td>\n",
       "      <td>['ZEN']</td>\n",
       "      <td>If an opponent drew three or more cards this t...</td>\n",
       "      <td>Instant — Trap</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZEN</td>\n",
       "      <td>Zendikar</td>\n",
       "      <td>aff3807b-9aa9-5c0b-b45a-2c6dc6f3bb5f</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2325</th>\n",
       "      <td>22518</td>\n",
       "      <td>66037</td>\n",
       "      <td>Greenweaver Druid</td>\n",
       "      <td>185694.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}{G}</td>\n",
       "      <td>3.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>['DPA', 'PS11', 'ZEN']</td>\n",
       "      <td>{T}: Add {G}{G} to your mana pool.</td>\n",
       "      <td>Creature — Elf Druid</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZEN</td>\n",
       "      <td>Zendikar</td>\n",
       "      <td>fdb17ac5-bbfd-5596-90ed-2fede299ce3e</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2326</th>\n",
       "      <td>22532</td>\n",
       "      <td>66078</td>\n",
       "      <td>Khalni Gem</td>\n",
       "      <td>198519.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{4}</td>\n",
       "      <td>4.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>['ZEN']</td>\n",
       "      <td>When Khalni Gem enters the battlefield, return...</td>\n",
       "      <td>Artifact</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZEN</td>\n",
       "      <td>Zendikar</td>\n",
       "      <td>6459301e-638e-56dc-9dd4-8b98be4f9baf</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2327</th>\n",
       "      <td>22537</td>\n",
       "      <td>66145</td>\n",
       "      <td>Trove Warden</td>\n",
       "      <td>495895.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}{W}{W}</td>\n",
       "      <td>4.0</td>\n",
       "      <td>['White']</td>\n",
       "      <td>['W']</td>\n",
       "      <td>...</td>\n",
       "      <td>['ZNC']</td>\n",
       "      <td>Vigilance\\nLandfall — Whenever a land enters t...</td>\n",
       "      <td>Creature — Cat Beast</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZNC</td>\n",
       "      <td>Zendikar Rising Commander</td>\n",
       "      <td>effc5bdb-2f8b-5d9a-b057-63cff0cf4b63</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2328</th>\n",
       "      <td>22539</td>\n",
       "      <td>66147</td>\n",
       "      <td>Whispersteel Dagger</td>\n",
       "      <td>495897.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}{B}</td>\n",
       "      <td>3.0</td>\n",
       "      <td>['Black']</td>\n",
       "      <td>['B']</td>\n",
       "      <td>...</td>\n",
       "      <td>['PLIST', 'ZNC']</td>\n",
       "      <td>Equipped creature gets +2/+0.\\nWhenever equipp...</td>\n",
       "      <td>Artifact — Equipment</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZNC</td>\n",
       "      <td>Zendikar Rising Commander</td>\n",
       "      <td>e923979f-f908-550d-88cb-4f8c27ce447d</td>\n",
       "      <td>True</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>2329 rows × 42 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "      level_0  index                   name  multiverse_id  layout  names  \\\n",
       "0          88    128  March of the Machines       106555.0  normal    NaN   \n",
       "1         128    184         Consume Spirit       129505.0  normal    NaN   \n",
       "2         209    294              Manabarbs       130367.0  normal    NaN   \n",
       "3         243    340      Birds of Paradise       129906.0  normal    NaN   \n",
       "4         259    364           Joiner Adept       130500.0  normal    NaN   \n",
       "...       ...    ...                    ...            ...     ...    ...   \n",
       "2324    22512  66019         Runeflare Trap       197536.0  normal    NaN   \n",
       "2325    22518  66037      Greenweaver Druid       185694.0  normal    NaN   \n",
       "2326    22532  66078             Khalni Gem       198519.0  normal    NaN   \n",
       "2327    22537  66145           Trove Warden       495895.0  normal    NaN   \n",
       "2328    22539  66147    Whispersteel Dagger       495897.0  normal    NaN   \n",
       "\n",
       "      mana_cost  cmc     colors color_identity  ...  \\\n",
       "0        {3}{U}  4.0   ['Blue']          ['U']  ...   \n",
       "1     {X}{1}{B}  2.0  ['Black']          ['B']  ...   \n",
       "2        {3}{R}  4.0    ['Red']          ['R']  ...   \n",
       "3           {G}  1.0  ['Green']          ['G']  ...   \n",
       "4        {1}{G}  2.0  ['Green']          ['G']  ...   \n",
       "...         ...  ...        ...            ...  ...   \n",
       "2324  {4}{R}{R}  6.0    ['Red']          ['R']  ...   \n",
       "2325     {2}{G}  3.0  ['Green']          ['G']  ...   \n",
       "2326        {4}  4.0        NaN            NaN  ...   \n",
       "2327  {2}{W}{W}  4.0  ['White']          ['W']  ...   \n",
       "2328     {2}{B}  3.0  ['Black']          ['B']  ...   \n",
       "\n",
       "                                              printings  \\\n",
       "0                                 ['10E', 'ARC', 'MRD']   \n",
       "1     ['10E', '9ED', 'DDC', 'DPA', 'DVD', 'HOP', 'M1...   \n",
       "2     ['10E', '2ED', '3ED', '4BB', '4ED', '5ED', '6E...   \n",
       "3     ['10E', '2ED', '3ED', '4BB', '4ED', '5ED', '6E...   \n",
       "4                                        ['10E', '5DN']   \n",
       "...                                                 ...   \n",
       "2324                                            ['ZEN']   \n",
       "2325                             ['DPA', 'PS11', 'ZEN']   \n",
       "2326                                            ['ZEN']   \n",
       "2327                                            ['ZNC']   \n",
       "2328                                   ['PLIST', 'ZNC']   \n",
       "\n",
       "                                          original_text         original_type  \\\n",
       "0     Each noncreature artifact is an artifact creat...           Enchantment   \n",
       "1     Spend only black mana on X.\\nConsume Spirit de...               Sorcery   \n",
       "2     Whenever a player taps a land for mana, Manaba...           Enchantment   \n",
       "3     Flying (This creature can't be blocked except ...       Creature - Bird   \n",
       "4     Lands you control have \"{T}: Add one mana of a...  Creature - Elf Druid   \n",
       "...                                                 ...                   ...   \n",
       "2324  If an opponent drew three or more cards this t...        Instant — Trap   \n",
       "2325                 {T}: Add {G}{G} to your mana pool.  Creature — Elf Druid   \n",
       "2326  When Khalni Gem enters the battlefield, return...              Artifact   \n",
       "2327  Vigilance\\nLandfall — Whenever a land enters t...  Creature — Cat Beast   \n",
       "2328  Equipped creature gets +2/+0.\\nWhenever equipp...  Artifact — Equipment   \n",
       "\n",
       "                                             legalities source  \\\n",
       "0     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "1     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "3     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "4     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "...                                                 ...    ...   \n",
       "2324  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2325  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2326  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2327  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2328  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "\n",
       "                                              image_url  set  \\\n",
       "0     http://gatherer.wizards.com/Handlers/Image.ash...  10E   \n",
       "1     http://gatherer.wizards.com/Handlers/Image.ash...  10E   \n",
       "2     http://gatherer.wizards.com/Handlers/Image.ash...  10E   \n",
       "3     http://gatherer.wizards.com/Handlers/Image.ash...  10E   \n",
       "4     http://gatherer.wizards.com/Handlers/Image.ash...  10E   \n",
       "...                                                 ...  ...   \n",
       "2324  http://gatherer.wizards.com/Handlers/Image.ash...  ZEN   \n",
       "2325  http://gatherer.wizards.com/Handlers/Image.ash...  ZEN   \n",
       "2326  http://gatherer.wizards.com/Handlers/Image.ash...  ZEN   \n",
       "2327  http://gatherer.wizards.com/Handlers/Image.ash...  ZNC   \n",
       "2328  http://gatherer.wizards.com/Handlers/Image.ash...  ZNC   \n",
       "\n",
       "                       set_name                                    id  \\\n",
       "0                 Tenth Edition  60961bae-4655-5a6b-843d-f6180c2c310d   \n",
       "1                 Tenth Edition  26180f1b-41a1-5fa5-9f6b-39ee08a1a203   \n",
       "2                 Tenth Edition  85245362-de1a-59fd-9f9a-d6a0c9522592   \n",
       "3                 Tenth Edition  28e56ac7-6a09-5118-9ade-6755b051bc0b   \n",
       "4                 Tenth Edition  2f39f144-a02a-5017-a45a-4a3cfc2c7a36   \n",
       "...                         ...                                   ...   \n",
       "2324                   Zendikar  aff3807b-9aa9-5c0b-b45a-2c6dc6f3bb5f   \n",
       "2325                   Zendikar  fdb17ac5-bbfd-5596-90ed-2fede299ce3e   \n",
       "2326                   Zendikar  6459301e-638e-56dc-9dd4-8b98be4f9baf   \n",
       "2327  Zendikar Rising Commander  effc5bdb-2f8b-5d9a-b057-63cff0cf4b63   \n",
       "2328  Zendikar Rising Commander  e923979f-f908-550d-88cb-4f8c27ce447d   \n",
       "\n",
       "     mana_is_in  \n",
       "0          True  \n",
       "1          True  \n",
       "2          True  \n",
       "3          True  \n",
       "4          True  \n",
       "...         ...  \n",
       "2324       True  \n",
       "2325       True  \n",
       "2326       True  \n",
       "2327       True  \n",
       "2328       True  \n",
       "\n",
       "[2329 rows x 42 columns]"
      ]
     },
     "execution_count": 138,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_mana_no_lands"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 147,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "'Add 3 black mana to your mana pool.'"
      ]
     },
     "execution_count": 147,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_mana_no_lands['original_text'][26]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 149,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "{'Add_mana': '3 black'}"
      ]
     },
     "execution_count": 149,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "parse (df_mana_no_lands['original_text'][26])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 150,
   "metadata": {},
   "outputs": [],
   "source": [
    "dict = parse (df_mana_no_lands['original_text'][11])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 142,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0"
      ]
     },
     "execution_count": 142,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "dict.get('Add_mana')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "code",
   "execution_count": 152,
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/opt/tljh/user/lib/python3.7/site-packages/ipykernel_launcher.py:5: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n",
      "/opt/tljh/user/lib/python3.7/site-packages/pandas/core/indexing.py:670: SettingWithCopyWarning:\n",
      "\n",
      "\n",
      "A value is trying to be set on a copy of a slice from a DataFrame\n",
      "\n",
      "See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy\n",
      "\n"
     ]
    }
   ],
   "source": [
    "df_mana_no_lands['new_mana'] = 0\n",
    "\n",
    "for i in range(len(df_mana_no_lands)):\n",
    "    dict = parse(df_mana_no_lands.original_text[i])\n",
    "    df_mana_no_lands['new_mana'][i] = dict.get('Add_mana')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 162,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_mana_no_lands_mana = df_mana_no_lands.query('new_mana != 0')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 170,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>level_0</th>\n",
       "      <th>index</th>\n",
       "      <th>name</th>\n",
       "      <th>multiverse_id</th>\n",
       "      <th>layout</th>\n",
       "      <th>names</th>\n",
       "      <th>mana_cost</th>\n",
       "      <th>cmc</th>\n",
       "      <th>colors</th>\n",
       "      <th>color_identity</th>\n",
       "      <th>...</th>\n",
       "      <th>original_text</th>\n",
       "      <th>original_type</th>\n",
       "      <th>legalities</th>\n",
       "      <th>source</th>\n",
       "      <th>image_url</th>\n",
       "      <th>set</th>\n",
       "      <th>set_name</th>\n",
       "      <th>id</th>\n",
       "      <th>mana_is_in</th>\n",
       "      <th>new_mana</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>243</td>\n",
       "      <td>340</td>\n",
       "      <td>Birds of Paradise</td>\n",
       "      <td>129906.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{G}</td>\n",
       "      <td>1.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>Flying (This creature can't be blocked except ...</td>\n",
       "      <td>Creature - Bird</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>28e56ac7-6a09-5118-9ade-6755b051bc0b</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>259</td>\n",
       "      <td>364</td>\n",
       "      <td>Joiner Adept</td>\n",
       "      <td>130500.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{1}{G}</td>\n",
       "      <td>2.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>Lands you control have \"{T}: Add one mana of a...</td>\n",
       "      <td>Creature - Elf Druid</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>2f39f144-a02a-5017-a45a-4a3cfc2c7a36</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>262</td>\n",
       "      <td>367</td>\n",
       "      <td>Llanowar Elves</td>\n",
       "      <td>129626.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{G}</td>\n",
       "      <td>1.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>{T}: Add {G} to your mana pool.</td>\n",
       "      <td>Creature - Elf Druid</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>51106f17-5dd1-5853-b45b-453d83b9d979</td>\n",
       "      <td>True</td>\n",
       "      <td>{G} to your</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>301</td>\n",
       "      <td>424</td>\n",
       "      <td>Chromatic Star</td>\n",
       "      <td>135279.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{1}</td>\n",
       "      <td>1.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>{1}, {T}, Sacrifice Chromatic Star: Add one ma...</td>\n",
       "      <td>Artifact</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>3785490a-01f5-511d-b471-60b1209b3d4f</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>305</td>\n",
       "      <td>430</td>\n",
       "      <td>Composite Golem</td>\n",
       "      <td>135275.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{6}</td>\n",
       "      <td>6.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>['B', 'G', 'R', 'U', 'W']</td>\n",
       "      <td>...</td>\n",
       "      <td>Sacrifice Composite Golem: Add {W}{U}{B}{R}{G}...</td>\n",
       "      <td>Artifact Creature - Golem</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>10E</td>\n",
       "      <td>Tenth Edition</td>\n",
       "      <td>7db42d14-4950-5478-b87c-fbbd93889801</td>\n",
       "      <td>True</td>\n",
       "      <td>{W}{U}{B}{R}{G} to your</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2314</th>\n",
       "      <td>22426</td>\n",
       "      <td>65664</td>\n",
       "      <td>Pillar of Origins</td>\n",
       "      <td>435399.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}</td>\n",
       "      <td>2.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>As Pillar of Origins enters the battlefield, c...</td>\n",
       "      <td>Artifact</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>XLN</td>\n",
       "      <td>Ixalan</td>\n",
       "      <td>ef746800-e68e-56db-a035-c0c483bddb28</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2315</th>\n",
       "      <td>22428</td>\n",
       "      <td>65668</td>\n",
       "      <td>Prying Blade</td>\n",
       "      <td>435403.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{1}</td>\n",
       "      <td>1.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>Equipped creature gets +1/+0.\\nWhenever equipp...</td>\n",
       "      <td>Artifact — Equipment</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>XLN</td>\n",
       "      <td>Ixalan</td>\n",
       "      <td>f89d8a19-1028-504a-bc94-4ea487b8ca14</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2316</th>\n",
       "      <td>22433</td>\n",
       "      <td>65675</td>\n",
       "      <td>Treasure Map // Treasure Cove</td>\n",
       "      <td>435410.0</td>\n",
       "      <td>transform</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}</td>\n",
       "      <td>2.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>{1}, {T}: Scry 1. Put a landmark counter on Tr...</td>\n",
       "      <td>Artifact</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>XLN</td>\n",
       "      <td>Ixalan</td>\n",
       "      <td>27c48a7a-d2a2-5459-b13c-461fcea64473</td>\n",
       "      <td>True</td>\n",
       "      <td>one</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2325</th>\n",
       "      <td>22518</td>\n",
       "      <td>66037</td>\n",
       "      <td>Greenweaver Druid</td>\n",
       "      <td>185694.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{2}{G}</td>\n",
       "      <td>3.0</td>\n",
       "      <td>['Green']</td>\n",
       "      <td>['G']</td>\n",
       "      <td>...</td>\n",
       "      <td>{T}: Add {G}{G} to your mana pool.</td>\n",
       "      <td>Creature — Elf Druid</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZEN</td>\n",
       "      <td>Zendikar</td>\n",
       "      <td>fdb17ac5-bbfd-5596-90ed-2fede299ce3e</td>\n",
       "      <td>True</td>\n",
       "      <td>{G}{G} to your</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2326</th>\n",
       "      <td>22532</td>\n",
       "      <td>66078</td>\n",
       "      <td>Khalni Gem</td>\n",
       "      <td>198519.0</td>\n",
       "      <td>normal</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{4}</td>\n",
       "      <td>4.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>NaN</td>\n",
       "      <td>...</td>\n",
       "      <td>When Khalni Gem enters the battlefield, return...</td>\n",
       "      <td>Artifact</td>\n",
       "      <td>[{'format': 'Commander', 'legality': 'Legal'},...</td>\n",
       "      <td>NaN</td>\n",
       "      <td>http://gatherer.wizards.com/Handlers/Image.ash...</td>\n",
       "      <td>ZEN</td>\n",
       "      <td>Zendikar</td>\n",
       "      <td>6459301e-638e-56dc-9dd4-8b98be4f9baf</td>\n",
       "      <td>True</td>\n",
       "      <td>two</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>675 rows × 43 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "      level_0  index                           name  multiverse_id     layout  \\\n",
       "3         243    340              Birds of Paradise       129906.0     normal   \n",
       "4         259    364                   Joiner Adept       130500.0     normal   \n",
       "5         262    367                 Llanowar Elves       129626.0     normal   \n",
       "8         301    424                 Chromatic Star       135279.0     normal   \n",
       "10        305    430                Composite Golem       135275.0     normal   \n",
       "...       ...    ...                            ...            ...        ...   \n",
       "2314    22426  65664              Pillar of Origins       435399.0     normal   \n",
       "2315    22428  65668                   Prying Blade       435403.0     normal   \n",
       "2316    22433  65675  Treasure Map // Treasure Cove       435410.0  transform   \n",
       "2325    22518  66037              Greenweaver Druid       185694.0     normal   \n",
       "2326    22532  66078                     Khalni Gem       198519.0     normal   \n",
       "\n",
       "      names mana_cost  cmc     colors             color_identity  ...  \\\n",
       "3       NaN       {G}  1.0  ['Green']                      ['G']  ...   \n",
       "4       NaN    {1}{G}  2.0  ['Green']                      ['G']  ...   \n",
       "5       NaN       {G}  1.0  ['Green']                      ['G']  ...   \n",
       "8       NaN       {1}  1.0        NaN                        NaN  ...   \n",
       "10      NaN       {6}  6.0        NaN  ['B', 'G', 'R', 'U', 'W']  ...   \n",
       "...     ...       ...  ...        ...                        ...  ...   \n",
       "2314    NaN       {2}  2.0        NaN                        NaN  ...   \n",
       "2315    NaN       {1}  1.0        NaN                        NaN  ...   \n",
       "2316    NaN       {2}  2.0        NaN                        NaN  ...   \n",
       "2325    NaN    {2}{G}  3.0  ['Green']                      ['G']  ...   \n",
       "2326    NaN       {4}  4.0        NaN                        NaN  ...   \n",
       "\n",
       "                                          original_text  \\\n",
       "3     Flying (This creature can't be blocked except ...   \n",
       "4     Lands you control have \"{T}: Add one mana of a...   \n",
       "5                       {T}: Add {G} to your mana pool.   \n",
       "8     {1}, {T}, Sacrifice Chromatic Star: Add one ma...   \n",
       "10    Sacrifice Composite Golem: Add {W}{U}{B}{R}{G}...   \n",
       "...                                                 ...   \n",
       "2314  As Pillar of Origins enters the battlefield, c...   \n",
       "2315  Equipped creature gets +1/+0.\\nWhenever equipp...   \n",
       "2316  {1}, {T}: Scry 1. Put a landmark counter on Tr...   \n",
       "2325                 {T}: Add {G}{G} to your mana pool.   \n",
       "2326  When Khalni Gem enters the battlefield, return...   \n",
       "\n",
       "                  original_type  \\\n",
       "3               Creature - Bird   \n",
       "4          Creature - Elf Druid   \n",
       "5          Creature - Elf Druid   \n",
       "8                      Artifact   \n",
       "10    Artifact Creature - Golem   \n",
       "...                         ...   \n",
       "2314                   Artifact   \n",
       "2315       Artifact — Equipment   \n",
       "2316                   Artifact   \n",
       "2325       Creature — Elf Druid   \n",
       "2326                   Artifact   \n",
       "\n",
       "                                             legalities source  \\\n",
       "3     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "4     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "5     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "8     [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "10    [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "...                                                 ...    ...   \n",
       "2314  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2315  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2316  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2325  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "2326  [{'format': 'Commander', 'legality': 'Legal'},...    NaN   \n",
       "\n",
       "                                              image_url  set       set_name  \\\n",
       "3     http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "4     http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "5     http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "8     http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "10    http://gatherer.wizards.com/Handlers/Image.ash...  10E  Tenth Edition   \n",
       "...                                                 ...  ...            ...   \n",
       "2314  http://gatherer.wizards.com/Handlers/Image.ash...  XLN         Ixalan   \n",
       "2315  http://gatherer.wizards.com/Handlers/Image.ash...  XLN         Ixalan   \n",
       "2316  http://gatherer.wizards.com/Handlers/Image.ash...  XLN         Ixalan   \n",
       "2325  http://gatherer.wizards.com/Handlers/Image.ash...  ZEN       Zendikar   \n",
       "2326  http://gatherer.wizards.com/Handlers/Image.ash...  ZEN       Zendikar   \n",
       "\n",
       "                                        id mana_is_in                 new_mana  \n",
       "3     28e56ac7-6a09-5118-9ade-6755b051bc0b       True                      one  \n",
       "4     2f39f144-a02a-5017-a45a-4a3cfc2c7a36       True                      one  \n",
       "5     51106f17-5dd1-5853-b45b-453d83b9d979       True              {G} to your  \n",
       "8     3785490a-01f5-511d-b471-60b1209b3d4f       True                      one  \n",
       "10    7db42d14-4950-5478-b87c-fbbd93889801       True  {W}{U}{B}{R}{G} to your  \n",
       "...                                    ...        ...                      ...  \n",
       "2314  ef746800-e68e-56db-a035-c0c483bddb28       True                      one  \n",
       "2315  f89d8a19-1028-504a-bc94-4ea487b8ca14       True                      one  \n",
       "2316  27c48a7a-d2a2-5459-b13c-461fcea64473       True                      one  \n",
       "2325  fdb17ac5-bbfd-5596-90ed-2fede299ce3e       True           {G}{G} to your  \n",
       "2326  6459301e-638e-56dc-9dd4-8b98be4f9baf       True                      two  \n",
       "\n",
       "[675 rows x 43 columns]"
      ]
     },
     "execution_count": 170,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_mana_no_lands_mana"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 193,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "2.994543276695799"
      ]
     },
     "execution_count": 193,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_mana_no_lands_mana.mana_is_in.count() / df_mana.mana_is_in.count() *100"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# 3% карт, не являющимися землями дают ману с помощью своего эффекта"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 188,
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "df_pic = df_mana_no_lands_mana.groupby('new_mana', as_index = False).agg('index').count()\\\n",
    "    .sort_values('index', ascending = False)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 189,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>new_mana</th>\n",
       "      <th>index</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>27</th>\n",
       "      <td>one</td>\n",
       "      <td>241</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>49</th>\n",
       "      <td>{1} to your</td>\n",
       "      <td>50</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>71</th>\n",
       "      <td>{G} to your</td>\n",
       "      <td>35</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>63</th>\n",
       "      <td>{C} to your</td>\n",
       "      <td>28</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>45</th>\n",
       "      <td>two</td>\n",
       "      <td>25</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>38</th>\n",
       "      <td>that much</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>37</th>\n",
       "      <td>that creature's casting cost in any combinatio...</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>36</th>\n",
       "      <td>ten</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>35</th>\n",
       "      <td>or even. Exile each creature with converted</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>134</th>\n",
       "      <td>{∞} to your</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>135 rows × 2 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "                                              new_mana  index\n",
       "27                                                 one    241\n",
       "49                                         {1} to your     50\n",
       "71                                         {G} to your     35\n",
       "63                                         {C} to your     28\n",
       "45                                                 two     25\n",
       "..                                                 ...    ...\n",
       "38                                           that much      1\n",
       "37   that creature's casting cost in any combinatio...      1\n",
       "36                                                 ten      1\n",
       "35         or even. Exile each creature with converted      1\n",
       "134                                        {∞} to your      1\n",
       "\n",
       "[135 rows x 2 columns]"
      ]
     },
     "execution_count": 189,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df_pic"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 190,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "application/vnd.plotly.v1+json": {
       "config": {
        "plotlyServerURL": "https://plot.ly"
       },
       "data": [
        {
         "alignmentgroup": "True",
         "hovertemplate": "new_mana=%{x}<br>index=%{y}<extra></extra>",
         "legendgroup": "",
         "marker": {
          "color": "#636efa"
         },
         "name": "",
         "offsetgroup": "",
         "orientation": "v",
         "showlegend": false,
         "textposition": "auto",
         "type": "bar",
         "x": [
          "one",
          "{1} to your",
          "{G} to your",
          "{C} to your",
          "two",
          "{R} to your",
          "X",
          "three",
          "{2} to your",
          "{B} to your",
          "{W} or {U} to your",
          "{B} or {R} to your",
          "to your",
          "{G} or {W} to your",
          "{R} or {G} to your",
          "{U} or {B} to your",
          "one colorless",
          "{R}{R}{R} to your",
          "{G}{G} to your",
          "{G} or {U} to your",
          "{R}{R} to your",
          "{B}{B}{B} to your",
          "two colorless",
          "{U} to your",
          "{C}. Spend this",
          "{B} or {G} to your",
          "{W} to your",
          "{W}{U}{B}{R}{G} to your",
          "{C}{C}. Spend this",
          "{3} to your",
          "one green",
          "{U} or {R} to your",
          "{W} or {B} to your",
          "that much {G} to your",
          "{R} or {W} to your",
          "{G}, {U}, or {R} to your",
          "an amount of {G} to your",
          "{B}{B} to your",
          "{U}. Spend this",
          "{X} to your",
          "{G}{G}{G} to your",
          "{U}{R} to your",
          "{R}, {G}, or {W} to your",
          "{U}, {B}, or {R} to your",
          "3 colorless",
          "{R} or {G}. If that",
          "{R} or {G}. Until end of turn, you don't lose this",
          "{R}. Spend this",
          "{R}{G}{W} to your",
          "{R}{G} to your",
          "{R}. When that",
          "{R}. Until end of turn, you don't lose this",
          "{R}, {W}, or {B} to your",
          "1",
          "{R}{R}{R}{R}{R}{R}{R} to your",
          "{R}{R}{R}. Spend this",
          "{R}{R}{R}{G}{G}{G}. Until end of turn, you don't lose this",
          "{W}{W}{W} and you gain 3 life. Until end of turn, you don't lose this",
          "{W}{W}{U}{U}{B}{B}{R}{R}{G}{G}. Spend this",
          "{W}{W}{U}{U}{B}{B}{R}{R}{G}{G} to your",
          "{W}{U}{B}{R}{G}. When you cast your next spell this turn, exile cards from the top of your library until you exile an instant or sorcery card with lesser",
          "{W}{U}{B}{R}{G}. This",
          "{W}{U}. Spend this",
          "{W}{U} to your",
          "{W}{B} to your",
          "{W}. Spend this",
          "{W}, {U}, or {B} to your",
          "{W}, {B}, or {G} to your",
          "{U}{B}{R}. Spend this",
          "{U}{B}{R} to your",
          "{U}{B} to your",
          "{U}, {R}, or {W}. Spend this",
          "{U}, {R}, or {W} to your",
          "{U} or {R}. Spend this",
          "{R}{W}{B} to your",
          "{R}{W} to your",
          "{R}{R}{R}{R}{R}{R}{R}{R} to your",
          "{R} for each Berserker you control. Until end of turn, you don't lose this",
          "{R}{R}{R}{R}{R} to your",
          "{R}{R}{R}{R} to your",
          "{R} for each card in target opponent's hand. Until end of turn, you don't lose this",
          "{C}{C}{C}{C} to your",
          "{HR} to your",
          "one blue",
          "one additional",
          "o2 to your",
          "o1 to your",
          "mana, instead all players add that",
          "mana of the type and amount last used to put a charge counter on Ice Cauldron to your",
          "mana equal to enchanted permanent's",
          "four",
          "converted",
          "colorless",
          "an equal amount of colorless",
          "an amount of {G} equal to its power. Until end of turn, you don't lose this",
          "an amount of {C} equal to that spell's converted",
          "an amount of colorless",
          "an amount of black",
          "an amount of",
          "X plus one colorless",
          "3 black",
          "2 colorless",
          "2",
          "1 white",
          "1 red",
          "1 green",
          "1 blue",
          "one black",
          "one red",
          "{G}{W} to your",
          "one white",
          "{G}{U} to your",
          "{G}{G}. Spend this",
          "{G}{G} to that player's",
          "{G}. When you spend this",
          "{G}, {W}, or {U} to your",
          "{C}{G}{U} to your",
          "1 black",
          "{C}{C} to your",
          "{C} for each {S} spent to cast this spell. Until end of turn, you don't lose this",
          "{B}{G} to your",
          "{B}{B}{B}{B} to your",
          "{B}, {R}, or {G} to your",
          "{B}, {G}, or {U} to your",
          "{2}{R} to your",
          "up to two colorless",
          "up to X",
          "three colorless",
          "the total",
          "that much {R}. Until end of turn, you don't lose this",
          "that much",
          "that creature's casting cost in any combination of red and/or black",
          "ten",
          "or even. Exile each creature with converted",
          "{∞} to your"
         ],
         "xaxis": "x",
         "y": [
          241,
          50,
          35,
          28,
          25,
          19,
          16,
          11,
          10,
          10,
          8,
          8,
          8,
          7,
          7,
          7,
          6,
          6,
          6,
          5,
          5,
          4,
          4,
          4,
          4,
          4,
          4,
          4,
          3,
          3,
          3,
          3,
          3,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          2,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1,
          1
         ],
         "yaxis": "y"
        }
       ],
       "layout": {
        "barmode": "relative",
        "legend": {
         "tracegroupgap": 0
        },
        "margin": {
         "t": 60
        },
        "template": {
         "data": {
          "bar": [
           {
            "error_x": {
             "color": "#2a3f5f"
            },
            "error_y": {
             "color": "#2a3f5f"
            },
            "marker": {
             "line": {
              "color": "#E5ECF6",
              "width": 0.5
             }
            },
            "type": "bar"
           }
          ],
          "barpolar": [
           {
            "marker": {
             "line": {
              "color": "#E5ECF6",
              "width": 0.5
             }
            },
            "type": "barpolar"
           }
          ],
          "carpet": [
           {
            "aaxis": {
             "endlinecolor": "#2a3f5f",
             "gridcolor": "white",
             "linecolor": "white",
             "minorgridcolor": "white",
             "startlinecolor": "#2a3f5f"
            },
            "baxis": {
             "endlinecolor": "#2a3f5f",
             "gridcolor": "white",
             "linecolor": "white",
             "minorgridcolor": "white",
             "startlinecolor": "#2a3f5f"
            },
            "type": "carpet"
           }
          ],
          "choropleth": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "choropleth"
           }
          ],
          "contour": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "contour"
           }
          ],
          "contourcarpet": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "contourcarpet"
           }
          ],
          "heatmap": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "heatmap"
           }
          ],
          "heatmapgl": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "heatmapgl"
           }
          ],
          "histogram": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "histogram"
           }
          ],
          "histogram2d": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "histogram2d"
           }
          ],
          "histogram2dcontour": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "histogram2dcontour"
           }
          ],
          "mesh3d": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "mesh3d"
           }
          ],
          "parcoords": [
           {
            "line": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "parcoords"
           }
          ],
          "pie": [
           {
            "automargin": true,
            "type": "pie"
           }
          ],
          "scatter": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatter"
           }
          ],
          "scatter3d": [
           {
            "line": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatter3d"
           }
          ],
          "scattercarpet": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattercarpet"
           }
          ],
          "scattergeo": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattergeo"
           }
          ],
          "scattergl": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattergl"
           }
          ],
          "scattermapbox": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattermapbox"
           }
          ],
          "scatterpolar": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterpolar"
           }
          ],
          "scatterpolargl": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterpolargl"
           }
          ],
          "scatterternary": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterternary"
           }
          ],
          "surface": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "surface"
           }
          ],
          "table": [
           {
            "cells": {
             "fill": {
              "color": "#EBF0F8"
             },
             "line": {
              "color": "white"
             }
            },
            "header": {
             "fill": {
              "color": "#C8D4E3"
             },
             "line": {
              "color": "white"
             }
            },
            "type": "table"
           }
          ]
         },
         "layout": {
          "annotationdefaults": {
           "arrowcolor": "#2a3f5f",
           "arrowhead": 0,
           "arrowwidth": 1
          },
          "coloraxis": {
           "colorbar": {
            "outlinewidth": 0,
            "ticks": ""
           }
          },
          "colorscale": {
           "diverging": [
            [
             0,
             "#8e0152"
            ],
            [
             0.1,
             "#c51b7d"
            ],
            [
             0.2,
             "#de77ae"
            ],
            [
             0.3,
             "#f1b6da"
            ],
            [
             0.4,
             "#fde0ef"
            ],
            [
             0.5,
             "#f7f7f7"
            ],
            [
             0.6,
             "#e6f5d0"
            ],
            [
             0.7,
             "#b8e186"
            ],
            [
             0.8,
             "#7fbc41"
            ],
            [
             0.9,
             "#4d9221"
            ],
            [
             1,
             "#276419"
            ]
           ],
           "sequential": [
            [
             0,
             "#0d0887"
            ],
            [
             0.1111111111111111,
             "#46039f"
            ],
            [
             0.2222222222222222,
             "#7201a8"
            ],
            [
             0.3333333333333333,
             "#9c179e"
            ],
            [
             0.4444444444444444,
             "#bd3786"
            ],
            [
             0.5555555555555556,
             "#d8576b"
            ],
            [
             0.6666666666666666,
             "#ed7953"
            ],
            [
             0.7777777777777778,
             "#fb9f3a"
            ],
            [
             0.8888888888888888,
             "#fdca26"
            ],
            [
             1,
             "#f0f921"
            ]
           ],
           "sequentialminus": [
            [
             0,
             "#0d0887"
            ],
            [
             0.1111111111111111,
             "#46039f"
            ],
            [
             0.2222222222222222,
             "#7201a8"
            ],
            [
             0.3333333333333333,
             "#9c179e"
            ],
            [
             0.4444444444444444,
             "#bd3786"
            ],
            [
             0.5555555555555556,
             "#d8576b"
            ],
            [
             0.6666666666666666,
             "#ed7953"
            ],
            [
             0.7777777777777778,
             "#fb9f3a"
            ],
            [
             0.8888888888888888,
             "#fdca26"
            ],
            [
             1,
             "#f0f921"
            ]
           ]
          },
          "colorway": [
           "#636efa",
           "#EF553B",
           "#00cc96",
           "#ab63fa",
           "#FFA15A",
           "#19d3f3",
           "#FF6692",
           "#B6E880",
           "#FF97FF",
           "#FECB52"
          ],
          "font": {
           "color": "#2a3f5f"
          },
          "geo": {
           "bgcolor": "white",
           "lakecolor": "white",
           "landcolor": "#E5ECF6",
           "showlakes": true,
           "showland": true,
           "subunitcolor": "white"
          },
          "hoverlabel": {
           "align": "left"
          },
          "hovermode": "closest",
          "mapbox": {
           "style": "light"
          },
          "paper_bgcolor": "white",
          "plot_bgcolor": "#E5ECF6",
          "polar": {
           "angularaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "bgcolor": "#E5ECF6",
           "radialaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           }
          },
          "scene": {
           "xaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           },
           "yaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           },
           "zaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           }
          },
          "shapedefaults": {
           "line": {
            "color": "#2a3f5f"
           }
          },
          "ternary": {
           "aaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "baxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "bgcolor": "#E5ECF6",
           "caxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           }
          },
          "title": {
           "x": 0.05
          },
          "xaxis": {
           "automargin": true,
           "gridcolor": "white",
           "linecolor": "white",
           "ticks": "",
           "title": {
            "standoff": 15
           },
           "zerolinecolor": "white",
           "zerolinewidth": 2
          },
          "yaxis": {
           "automargin": true,
           "gridcolor": "white",
           "linecolor": "white",
           "ticks": "",
           "title": {
            "standoff": 15
           },
           "zerolinecolor": "white",
           "zerolinewidth": 2
          }
         }
        },
        "xaxis": {
         "anchor": "y",
         "domain": [
          0,
          1
         ],
         "title": {
          "text": "new_mana"
         }
        },
        "yaxis": {
         "anchor": "x",
         "domain": [
          0,
          1
         ],
         "title": {
          "text": "index"
         }
        }
       }
      },
      "text/html": [
       "<div>\n",
       "        \n",
       "        \n",
       "            <div id=\"6fe4cf56-4dc9-4ba0-9cbd-a6c57db211f8\" class=\"plotly-graph-div\" style=\"height:525px; width:100%;\"></div>\n",
       "            <script type=\"text/javascript\">\n",
       "                require([\"plotly\"], function(Plotly) {\n",
       "                    window.PLOTLYENV=window.PLOTLYENV || {};\n",
       "                    \n",
       "                if (document.getElementById(\"6fe4cf56-4dc9-4ba0-9cbd-a6c57db211f8\")) {\n",
       "                    Plotly.newPlot(\n",
       "                        '6fe4cf56-4dc9-4ba0-9cbd-a6c57db211f8',\n",
       "                        [{\"alignmentgroup\": \"True\", \"hovertemplate\": \"new_mana=%{x}<br>index=%{y}<extra></extra>\", \"legendgroup\": \"\", \"marker\": {\"color\": \"#636efa\"}, \"name\": \"\", \"offsetgroup\": \"\", \"orientation\": \"v\", \"showlegend\": false, \"textposition\": \"auto\", \"type\": \"bar\", \"x\": [\"one\", \"{1} to your\", \"{G} to your\", \"{C} to your\", \"two\", \"{R} to your\", \"X\", \"three\", \"{2} to your\", \"{B} to your\", \"{W} or {U} to your\", \"{B} or {R} to your\", \"to your\", \"{G} or {W} to your\", \"{R} or {G} to your\", \"{U} or {B} to your\", \"one colorless\", \"{R}{R}{R} to your\", \"{G}{G} to your\", \"{G} or {U} to your\", \"{R}{R} to your\", \"{B}{B}{B} to your\", \"two colorless\", \"{U} to your\", \"{C}. Spend this\", \"{B} or {G} to your\", \"{W} to your\", \"{W}{U}{B}{R}{G} to your\", \"{C}{C}. Spend this\", \"{3} to your\", \"one green\", \"{U} or {R} to your\", \"{W} or {B} to your\", \"that much {G} to your\", \"{R} or {W} to your\", \"{G}, {U}, or {R} to your\", \"an amount of {G} to your\", \"{B}{B} to your\", \"{U}. Spend this\", \"{X} to your\", \"{G}{G}{G} to your\", \"{U}{R} to your\", \"{R}, {G}, or {W} to your\", \"{U}, {B}, or {R} to your\", \"3 colorless\", \"{R} or {G}. If that\", \"{R} or {G}. Until end of turn, you don't lose this\", \"{R}. Spend this\", \"{R}{G}{W} to your\", \"{R}{G} to your\", \"{R}. When that\", \"{R}. Until end of turn, you don't lose this\", \"{R}, {W}, or {B} to your\", \"1\", \"{R}{R}{R}{R}{R}{R}{R} to your\", \"{R}{R}{R}. Spend this\", \"{R}{R}{R}{G}{G}{G}. Until end of turn, you don't lose this\", \"{W}{W}{W} and you gain 3 life. Until end of turn, you don't lose this\", \"{W}{W}{U}{U}{B}{B}{R}{R}{G}{G}. Spend this\", \"{W}{W}{U}{U}{B}{B}{R}{R}{G}{G} to your\", \"{W}{U}{B}{R}{G}. When you cast your next spell this turn, exile cards from the top of your library until you exile an instant or sorcery card with lesser\", \"{W}{U}{B}{R}{G}. This\", \"{W}{U}. Spend this\", \"{W}{U} to your\", \"{W}{B} to your\", \"{W}. Spend this\", \"{W}, {U}, or {B} to your\", \"{W}, {B}, or {G} to your\", \"{U}{B}{R}. Spend this\", \"{U}{B}{R} to your\", \"{U}{B} to your\", \"{U}, {R}, or {W}. Spend this\", \"{U}, {R}, or {W} to your\", \"{U} or {R}. Spend this\", \"{R}{W}{B} to your\", \"{R}{W} to your\", \"{R}{R}{R}{R}{R}{R}{R}{R} to your\", \"{R} for each Berserker you control. Until end of turn, you don't lose this\", \"{R}{R}{R}{R}{R} to your\", \"{R}{R}{R}{R} to your\", \"{R} for each card in target opponent's hand. Until end of turn, you don't lose this\", \"{C}{C}{C}{C} to your\", \"{HR} to your\", \"one blue\", \"one additional\", \"o2 to your\", \"o1 to your\", \"mana, instead all players add that\", \"mana of the type and amount last used to put a charge counter on Ice Cauldron to your\", \"mana equal to enchanted permanent's\", \"four\", \"converted\", \"colorless\", \"an equal amount of colorless\", \"an amount of {G} equal to its power. Until end of turn, you don't lose this\", \"an amount of {C} equal to that spell's converted\", \"an amount of colorless\", \"an amount of black\", \"an amount of\", \"X plus one colorless\", \"3 black\", \"2 colorless\", \"2\", \"1 white\", \"1 red\", \"1 green\", \"1 blue\", \"one black\", \"one red\", \"{G}{W} to your\", \"one white\", \"{G}{U} to your\", \"{G}{G}. Spend this\", \"{G}{G} to that player's\", \"{G}. When you spend this\", \"{G}, {W}, or {U} to your\", \"{C}{G}{U} to your\", \"1 black\", \"{C}{C} to your\", \"{C} for each {S} spent to cast this spell. Until end of turn, you don't lose this\", \"{B}{G} to your\", \"{B}{B}{B}{B} to your\", \"{B}, {R}, or {G} to your\", \"{B}, {G}, or {U} to your\", \"{2}{R} to your\", \"up to two colorless\", \"up to X\", \"three colorless\", \"the total\", \"that much {R}. Until end of turn, you don't lose this\", \"that much\", \"that creature's casting cost in any combination of red and/or black\", \"ten\", \"or even. Exile each creature with converted\", \"{\\u221e} to your\"], \"xaxis\": \"x\", \"y\": [241, 50, 35, 28, 25, 19, 16, 11, 10, 10, 8, 8, 8, 7, 7, 7, 6, 6, 6, 5, 5, 4, 4, 4, 4, 4, 4, 4, 3, 3, 3, 3, 3, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], \"yaxis\": \"y\"}],\n",
       "                        {\"barmode\": \"relative\", \"legend\": {\"tracegroupgap\": 0}, \"margin\": {\"t\": 60}, \"template\": {\"data\": {\"bar\": [{\"error_x\": {\"color\": \"#2a3f5f\"}, \"error_y\": {\"color\": \"#2a3f5f\"}, \"marker\": {\"line\": {\"color\": \"#E5ECF6\", \"width\": 0.5}}, \"type\": \"bar\"}], \"barpolar\": [{\"marker\": {\"line\": {\"color\": \"#E5ECF6\", \"width\": 0.5}}, \"type\": \"barpolar\"}], \"carpet\": [{\"aaxis\": {\"endlinecolor\": \"#2a3f5f\", \"gridcolor\": \"white\", \"linecolor\": \"white\", \"minorgridcolor\": \"white\", \"startlinecolor\": \"#2a3f5f\"}, \"baxis\": {\"endlinecolor\": \"#2a3f5f\", \"gridcolor\": \"white\", \"linecolor\": \"white\", \"minorgridcolor\": \"white\", \"startlinecolor\": \"#2a3f5f\"}, \"type\": \"carpet\"}], \"choropleth\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"type\": \"choropleth\"}], \"contour\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"contour\"}], \"contourcarpet\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"type\": \"contourcarpet\"}], \"heatmap\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"heatmap\"}], \"heatmapgl\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"heatmapgl\"}], \"histogram\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"histogram\"}], \"histogram2d\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"histogram2d\"}], \"histogram2dcontour\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"histogram2dcontour\"}], \"mesh3d\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"type\": \"mesh3d\"}], \"parcoords\": [{\"line\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"parcoords\"}], \"pie\": [{\"automargin\": true, \"type\": \"pie\"}], \"scatter\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scatter\"}], \"scatter3d\": [{\"line\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scatter3d\"}], \"scattercarpet\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scattercarpet\"}], \"scattergeo\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scattergeo\"}], \"scattergl\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scattergl\"}], \"scattermapbox\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scattermapbox\"}], \"scatterpolar\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scatterpolar\"}], \"scatterpolargl\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scatterpolargl\"}], \"scatterternary\": [{\"marker\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"type\": \"scatterternary\"}], \"surface\": [{\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}, \"colorscale\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"type\": \"surface\"}], \"table\": [{\"cells\": {\"fill\": {\"color\": \"#EBF0F8\"}, \"line\": {\"color\": \"white\"}}, \"header\": {\"fill\": {\"color\": \"#C8D4E3\"}, \"line\": {\"color\": \"white\"}}, \"type\": \"table\"}]}, \"layout\": {\"annotationdefaults\": {\"arrowcolor\": \"#2a3f5f\", \"arrowhead\": 0, \"arrowwidth\": 1}, \"coloraxis\": {\"colorbar\": {\"outlinewidth\": 0, \"ticks\": \"\"}}, \"colorscale\": {\"diverging\": [[0, \"#8e0152\"], [0.1, \"#c51b7d\"], [0.2, \"#de77ae\"], [0.3, \"#f1b6da\"], [0.4, \"#fde0ef\"], [0.5, \"#f7f7f7\"], [0.6, \"#e6f5d0\"], [0.7, \"#b8e186\"], [0.8, \"#7fbc41\"], [0.9, \"#4d9221\"], [1, \"#276419\"]], \"sequential\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]], \"sequentialminus\": [[0.0, \"#0d0887\"], [0.1111111111111111, \"#46039f\"], [0.2222222222222222, \"#7201a8\"], [0.3333333333333333, \"#9c179e\"], [0.4444444444444444, \"#bd3786\"], [0.5555555555555556, \"#d8576b\"], [0.6666666666666666, \"#ed7953\"], [0.7777777777777778, \"#fb9f3a\"], [0.8888888888888888, \"#fdca26\"], [1.0, \"#f0f921\"]]}, \"colorway\": [\"#636efa\", \"#EF553B\", \"#00cc96\", \"#ab63fa\", \"#FFA15A\", \"#19d3f3\", \"#FF6692\", \"#B6E880\", \"#FF97FF\", \"#FECB52\"], \"font\": {\"color\": \"#2a3f5f\"}, \"geo\": {\"bgcolor\": \"white\", \"lakecolor\": \"white\", \"landcolor\": \"#E5ECF6\", \"showlakes\": true, \"showland\": true, \"subunitcolor\": \"white\"}, \"hoverlabel\": {\"align\": \"left\"}, \"hovermode\": \"closest\", \"mapbox\": {\"style\": \"light\"}, \"paper_bgcolor\": \"white\", \"plot_bgcolor\": \"#E5ECF6\", \"polar\": {\"angularaxis\": {\"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\"}, \"bgcolor\": \"#E5ECF6\", \"radialaxis\": {\"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\"}}, \"scene\": {\"xaxis\": {\"backgroundcolor\": \"#E5ECF6\", \"gridcolor\": \"white\", \"gridwidth\": 2, \"linecolor\": \"white\", \"showbackground\": true, \"ticks\": \"\", \"zerolinecolor\": \"white\"}, \"yaxis\": {\"backgroundcolor\": \"#E5ECF6\", \"gridcolor\": \"white\", \"gridwidth\": 2, \"linecolor\": \"white\", \"showbackground\": true, \"ticks\": \"\", \"zerolinecolor\": \"white\"}, \"zaxis\": {\"backgroundcolor\": \"#E5ECF6\", \"gridcolor\": \"white\", \"gridwidth\": 2, \"linecolor\": \"white\", \"showbackground\": true, \"ticks\": \"\", \"zerolinecolor\": \"white\"}}, \"shapedefaults\": {\"line\": {\"color\": \"#2a3f5f\"}}, \"ternary\": {\"aaxis\": {\"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\"}, \"baxis\": {\"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\"}, \"bgcolor\": \"#E5ECF6\", \"caxis\": {\"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\"}}, \"title\": {\"x\": 0.05}, \"xaxis\": {\"automargin\": true, \"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\", \"title\": {\"standoff\": 15}, \"zerolinecolor\": \"white\", \"zerolinewidth\": 2}, \"yaxis\": {\"automargin\": true, \"gridcolor\": \"white\", \"linecolor\": \"white\", \"ticks\": \"\", \"title\": {\"standoff\": 15}, \"zerolinecolor\": \"white\", \"zerolinewidth\": 2}}}, \"xaxis\": {\"anchor\": \"y\", \"domain\": [0.0, 1.0], \"title\": {\"text\": \"new_mana\"}}, \"yaxis\": {\"anchor\": \"x\", \"domain\": [0.0, 1.0], \"title\": {\"text\": \"index\"}}},\n",
       "                        {\"responsive\": true}\n",
       "                    ).then(function(){\n",
       "                            \n",
       "var gd = document.getElementById('6fe4cf56-4dc9-4ba0-9cbd-a6c57db211f8');\n",
       "var x = new MutationObserver(function (mutations, observer) {{\n",
       "        var display = window.getComputedStyle(gd).display;\n",
       "        if (!display || display === 'none') {{\n",
       "            console.log([gd, 'removed!']);\n",
       "            Plotly.purge(gd);\n",
       "            observer.disconnect();\n",
       "        }}\n",
       "}});\n",
       "\n",
       "// Listen for the removal of the full notebook cells\n",
       "var notebookContainer = gd.closest('#notebook-container');\n",
       "if (notebookContainer) {{\n",
       "    x.observe(notebookContainer, {childList: true});\n",
       "}}\n",
       "\n",
       "// Listen for the clearing of the current output cell\n",
       "var outputEl = gd.closest('.output');\n",
       "if (outputEl) {{\n",
       "    x.observe(outputEl, {childList: true});\n",
       "}}\n",
       "\n",
       "                        })\n",
       "                };\n",
       "                });\n",
       "            </script>\n",
       "        </div>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "import plotly.express as px\n",
    "fig = px.bar(df_pic, x='new_mana', y='index')\n",
    "fig.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "из барплота видно, что наиболее часто среди карт, обладающих эффектом добавления маны, встречаются карты, которые добавляют 1 ману любого цвета "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.7.3"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
