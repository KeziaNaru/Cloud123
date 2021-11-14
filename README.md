{
 "cells": [
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "7dc2f0f0-7f2e-441b-ad5e-dc6c6e7e19b5",
    "_uuid": "944c75ecf0178c6158c015537796631f4be3ba67"
   },
   "outputs": [],
   "source": [
    "## Prediction of Breast Cancer using SVM with 99% accuracy"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "e06041ba-c4de-404d-b427-67773487cebb",
    "_uuid": "3ffdd62aeaec4f87cefa896ee8f46fba8ac12876"
   },
   "outputs": [],
   "source": [
    "Using the Breast Cancer Wisconsin (Diagnostic) Database, we can create a classifier that can help diagnose patients and predict the likelihood of a breast cancer. A few machine learning techniques will be explored. In this exercise, Support Vector Machine is being implemented with 99% accuracy."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {
    "_cell_guid": "5c7e9232-776b-4250-bc66-846d0211a8d6",
    "_execution_state": "idle",
    "_uuid": "44a9940e9fc2880db72ff036201617f1e2dcc2ac",
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "import numpy as np\n",
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "from sklearn.metrics import classification_report\n",
    "from sklearn.metrics import confusion_matrix\n",
    "from sklearn.metrics import accuracy_score\n",
    "from sklearn.model_selection import train_test_split\n",
    "from sklearn.model_selection import cross_val_score\n",
    "from sklearn.model_selection import KFold\n",
    "from sklearn.tree import DecisionTreeClassifier\n",
    "from sklearn.neighbors import KNeighborsClassifier\n",
    "from sklearn.naive_bayes import GaussianNB\n",
    "from sklearn.pipeline import Pipeline\n",
    "from sklearn.preprocessing import StandardScaler\n",
    "from sklearn.model_selection import GridSearchCV\n",
    "from sklearn.svm import SVC\n",
    "import time"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "10a5454a-2edb-4b71-95ce-cf3eb42ad08d",
    "_uuid": "9de448d38725cef68acd836b44c1f6cae2a280a4"
   },
   "outputs": [],
   "source": [
    "## Exploratory analysis\n",
    "\n",
    "Load the dataset and do some quick exploratory analysis."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {
    "_cell_guid": "991fcaa2-e4b6-4d1e-a480-c5335249546a",
    "_execution_state": "idle",
    "_uuid": "7175ebe2ad5a61144de6985dd44398f0c7778a76"
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style>\n",
       "    .dataframe thead tr:only-child th {\n",
       "        text-align: right;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: left;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>id</th>\n",
       "      <th>diagnosis</th>\n",
       "      <th>radius_mean</th>\n",
       "      <th>texture_mean</th>\n",
       "      <th>perimeter_mean</th>\n",
       "      <th>area_mean</th>\n",
       "      <th>smoothness_mean</th>\n",
       "      <th>compactness_mean</th>\n",
       "      <th>concavity_mean</th>\n",
       "      <th>concave points_mean</th>\n",
       "      <th>...</th>\n",
       "      <th>texture_worst</th>\n",
       "      <th>perimeter_worst</th>\n",
       "      <th>area_worst</th>\n",
       "      <th>smoothness_worst</th>\n",
       "      <th>compactness_worst</th>\n",
       "      <th>concavity_worst</th>\n",
       "      <th>concave points_worst</th>\n",
       "      <th>symmetry_worst</th>\n",
       "      <th>fractal_dimension_worst</th>\n",
       "      <th>Unnamed: 32</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>842302</td>\n",
       "      <td>M</td>\n",
       "      <td>17.99</td>\n",
       "      <td>10.38</td>\n",
       "      <td>122.80</td>\n",
       "      <td>1001.0</td>\n",
       "      <td>0.11840</td>\n",
       "      <td>0.27760</td>\n",
       "      <td>0.3001</td>\n",
       "      <td>0.14710</td>\n",
       "      <td>...</td>\n",
       "      <td>17.33</td>\n",
       "      <td>184.60</td>\n",
       "      <td>2019.0</td>\n",
       "      <td>0.1622</td>\n",
       "      <td>0.6656</td>\n",
       "      <td>0.7119</td>\n",
       "      <td>0.2654</td>\n",
       "      <td>0.4601</td>\n",
       "      <td>0.11890</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>842517</td>\n",
       "      <td>M</td>\n",
       "      <td>20.57</td>\n",
       "      <td>17.77</td>\n",
       "      <td>132.90</td>\n",
       "      <td>1326.0</td>\n",
       "      <td>0.08474</td>\n",
       "      <td>0.07864</td>\n",
       "      <td>0.0869</td>\n",
       "      <td>0.07017</td>\n",
       "      <td>...</td>\n",
       "      <td>23.41</td>\n",
       "      <td>158.80</td>\n",
       "      <td>1956.0</td>\n",
       "      <td>0.1238</td>\n",
       "      <td>0.1866</td>\n",
       "      <td>0.2416</td>\n",
       "      <td>0.1860</td>\n",
       "      <td>0.2750</td>\n",
       "      <td>0.08902</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>84300903</td>\n",
       "      <td>M</td>\n",
       "      <td>19.69</td>\n",
       "      <td>21.25</td>\n",
       "      <td>130.00</td>\n",
       "      <td>1203.0</td>\n",
       "      <td>0.10960</td>\n",
       "      <td>0.15990</td>\n",
       "      <td>0.1974</td>\n",
       "      <td>0.12790</td>\n",
       "      <td>...</td>\n",
       "      <td>25.53</td>\n",
       "      <td>152.50</td>\n",
       "      <td>1709.0</td>\n",
       "      <td>0.1444</td>\n",
       "      <td>0.4245</td>\n",
       "      <td>0.4504</td>\n",
       "      <td>0.2430</td>\n",
       "      <td>0.3613</td>\n",
       "      <td>0.08758</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>84348301</td>\n",
       "      <td>M</td>\n",
       "      <td>11.42</td>\n",
       "      <td>20.38</td>\n",
       "      <td>77.58</td>\n",
       "      <td>386.1</td>\n",
       "      <td>0.14250</td>\n",
       "      <td>0.28390</td>\n",
       "      <td>0.2414</td>\n",
       "      <td>0.10520</td>\n",
       "      <td>...</td>\n",
       "      <td>26.50</td>\n",
       "      <td>98.87</td>\n",
       "      <td>567.7</td>\n",
       "      <td>0.2098</td>\n",
       "      <td>0.8663</td>\n",
       "      <td>0.6869</td>\n",
       "      <td>0.2575</td>\n",
       "      <td>0.6638</td>\n",
       "      <td>0.17300</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>84358402</td>\n",
       "      <td>M</td>\n",
       "      <td>20.29</td>\n",
       "      <td>14.34</td>\n",
       "      <td>135.10</td>\n",
       "      <td>1297.0</td>\n",
       "      <td>0.10030</td>\n",
       "      <td>0.13280</td>\n",
       "      <td>0.1980</td>\n",
       "      <td>0.10430</td>\n",
       "      <td>...</td>\n",
       "      <td>16.67</td>\n",
       "      <td>152.20</td>\n",
       "      <td>1575.0</td>\n",
       "      <td>0.1374</td>\n",
       "      <td>0.2050</td>\n",
       "      <td>0.4000</td>\n",
       "      <td>0.1625</td>\n",
       "      <td>0.2364</td>\n",
       "      <td>0.07678</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>5 rows × 33 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "         id diagnosis  radius_mean  texture_mean  perimeter_mean  area_mean  \\\n",
       "0    842302         M        17.99         10.38          122.80     1001.0   \n",
       "1    842517         M        20.57         17.77          132.90     1326.0   \n",
       "2  84300903         M        19.69         21.25          130.00     1203.0   \n",
       "3  84348301         M        11.42         20.38           77.58      386.1   \n",
       "4  84358402         M        20.29         14.34          135.10     1297.0   \n",
       "\n",
       "   smoothness_mean  compactness_mean  concavity_mean  concave points_mean  \\\n",
       "0          0.11840           0.27760          0.3001              0.14710   \n",
       "1          0.08474           0.07864          0.0869              0.07017   \n",
       "2          0.10960           0.15990          0.1974              0.12790   \n",
       "3          0.14250           0.28390          0.2414              0.10520   \n",
       "4          0.10030           0.13280          0.1980              0.10430   \n",
       "\n",
       "      ...       texture_worst  perimeter_worst  area_worst  smoothness_worst  \\\n",
       "0     ...               17.33           184.60      2019.0            0.1622   \n",
       "1     ...               23.41           158.80      1956.0            0.1238   \n",
       "2     ...               25.53           152.50      1709.0            0.1444   \n",
       "3     ...               26.50            98.87       567.7            0.2098   \n",
       "4     ...               16.67           152.20      1575.0            0.1374   \n",
       "\n",
       "   compactness_worst  concavity_worst  concave points_worst  symmetry_worst  \\\n",
       "0             0.6656           0.7119                0.2654          0.4601   \n",
       "1             0.1866           0.2416                0.1860          0.2750   \n",
       "2             0.4245           0.4504                0.2430          0.3613   \n",
       "3             0.8663           0.6869                0.2575          0.6638   \n",
       "4             0.2050           0.4000                0.1625          0.2364   \n",
       "\n",
       "   fractal_dimension_worst  Unnamed: 32  \n",
       "0                  0.11890          NaN  \n",
       "1                  0.08902          NaN  \n",
       "2                  0.08758          NaN  \n",
       "3                  0.17300          NaN  \n",
       "4                  0.07678          NaN  \n",
       "\n",
       "[5 rows x 33 columns]"
      ]
     },
     "execution_count": 2,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "data = pd.read_csv('../input/data.csv', index_col=False)\n",
    "data.head(5)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {
    "_cell_guid": "0f5a9ef3-2b7e-40f3-94c9-d610c0f88dc5",
    "_uuid": "769147a9ae9ba27e572d07c2e75aa9b13287ee9b"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "(569, 33)\n"
     ]
    }
   ],
   "source": [
    "print(data.shape)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {
    "_cell_guid": "dfdb2cf8-9fe1-4efb-a779-9cb062a04c71",
    "_uuid": "54c91f25e50f6b069ed7217c34df68cd52cd5970"
   },
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style>\n",
       "    .dataframe thead tr:only-child th {\n",
       "        text-align: right;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: left;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>id</th>\n",
       "      <th>radius_mean</th>\n",
       "      <th>texture_mean</th>\n",
       "      <th>perimeter_mean</th>\n",
       "      <th>area_mean</th>\n",
       "      <th>smoothness_mean</th>\n",
       "      <th>compactness_mean</th>\n",
       "      <th>concavity_mean</th>\n",
       "      <th>concave points_mean</th>\n",
       "      <th>symmetry_mean</th>\n",
       "      <th>...</th>\n",
       "      <th>texture_worst</th>\n",
       "      <th>perimeter_worst</th>\n",
       "      <th>area_worst</th>\n",
       "      <th>smoothness_worst</th>\n",
       "      <th>compactness_worst</th>\n",
       "      <th>concavity_worst</th>\n",
       "      <th>concave points_worst</th>\n",
       "      <th>symmetry_worst</th>\n",
       "      <th>fractal_dimension_worst</th>\n",
       "      <th>Unnamed: 32</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>count</th>\n",
       "      <td>5.690000e+02</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>...</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>569.000000</td>\n",
       "      <td>0.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>mean</th>\n",
       "      <td>3.037183e+07</td>\n",
       "      <td>14.127292</td>\n",
       "      <td>19.289649</td>\n",
       "      <td>91.969033</td>\n",
       "      <td>654.889104</td>\n",
       "      <td>0.096360</td>\n",
       "      <td>0.104341</td>\n",
       "      <td>0.088799</td>\n",
       "      <td>0.048919</td>\n",
       "      <td>0.181162</td>\n",
       "      <td>...</td>\n",
       "      <td>25.677223</td>\n",
       "      <td>107.261213</td>\n",
       "      <td>880.583128</td>\n",
       "      <td>0.132369</td>\n",
       "      <td>0.254265</td>\n",
       "      <td>0.272188</td>\n",
       "      <td>0.114606</td>\n",
       "      <td>0.290076</td>\n",
       "      <td>0.083946</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>std</th>\n",
       "      <td>1.250206e+08</td>\n",
       "      <td>3.524049</td>\n",
       "      <td>4.301036</td>\n",
       "      <td>24.298981</td>\n",
       "      <td>351.914129</td>\n",
       "      <td>0.014064</td>\n",
       "      <td>0.052813</td>\n",
       "      <td>0.079720</td>\n",
       "      <td>0.038803</td>\n",
       "      <td>0.027414</td>\n",
       "      <td>...</td>\n",
       "      <td>6.146258</td>\n",
       "      <td>33.602542</td>\n",
       "      <td>569.356993</td>\n",
       "      <td>0.022832</td>\n",
       "      <td>0.157336</td>\n",
       "      <td>0.208624</td>\n",
       "      <td>0.065732</td>\n",
       "      <td>0.061867</td>\n",
       "      <td>0.018061</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>min</th>\n",
       "      <td>8.670000e+03</td>\n",
       "      <td>6.981000</td>\n",
       "      <td>9.710000</td>\n",
       "      <td>43.790000</td>\n",
       "      <td>143.500000</td>\n",
       "      <td>0.052630</td>\n",
       "      <td>0.019380</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.106000</td>\n",
       "      <td>...</td>\n",
       "      <td>12.020000</td>\n",
       "      <td>50.410000</td>\n",
       "      <td>185.200000</td>\n",
       "      <td>0.071170</td>\n",
       "      <td>0.027290</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.156500</td>\n",
       "      <td>0.055040</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25%</th>\n",
       "      <td>8.692180e+05</td>\n",
       "      <td>11.700000</td>\n",
       "      <td>16.170000</td>\n",
       "      <td>75.170000</td>\n",
       "      <td>420.300000</td>\n",
       "      <td>0.086370</td>\n",
       "      <td>0.064920</td>\n",
       "      <td>0.029560</td>\n",
       "      <td>0.020310</td>\n",
       "      <td>0.161900</td>\n",
       "      <td>...</td>\n",
       "      <td>21.080000</td>\n",
       "      <td>84.110000</td>\n",
       "      <td>515.300000</td>\n",
       "      <td>0.116600</td>\n",
       "      <td>0.147200</td>\n",
       "      <td>0.114500</td>\n",
       "      <td>0.064930</td>\n",
       "      <td>0.250400</td>\n",
       "      <td>0.071460</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>50%</th>\n",
       "      <td>9.060240e+05</td>\n",
       "      <td>13.370000</td>\n",
       "      <td>18.840000</td>\n",
       "      <td>86.240000</td>\n",
       "      <td>551.100000</td>\n",
       "      <td>0.095870</td>\n",
       "      <td>0.092630</td>\n",
       "      <td>0.061540</td>\n",
       "      <td>0.033500</td>\n",
       "      <td>0.179200</td>\n",
       "      <td>...</td>\n",
       "      <td>25.410000</td>\n",
       "      <td>97.660000</td>\n",
       "      <td>686.500000</td>\n",
       "      <td>0.131300</td>\n",
       "      <td>0.211900</td>\n",
       "      <td>0.226700</td>\n",
       "      <td>0.099930</td>\n",
       "      <td>0.282200</td>\n",
       "      <td>0.080040</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>75%</th>\n",
       "      <td>8.813129e+06</td>\n",
       "      <td>15.780000</td>\n",
       "      <td>21.800000</td>\n",
       "      <td>104.100000</td>\n",
       "      <td>782.700000</td>\n",
       "      <td>0.105300</td>\n",
       "      <td>0.130400</td>\n",
       "      <td>0.130700</td>\n",
       "      <td>0.074000</td>\n",
       "      <td>0.195700</td>\n",
       "      <td>...</td>\n",
       "      <td>29.720000</td>\n",
       "      <td>125.400000</td>\n",
       "      <td>1084.000000</td>\n",
       "      <td>0.146000</td>\n",
       "      <td>0.339100</td>\n",
       "      <td>0.382900</td>\n",
       "      <td>0.161400</td>\n",
       "      <td>0.317900</td>\n",
       "      <td>0.092080</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>max</th>\n",
       "      <td>9.113205e+08</td>\n",
       "      <td>28.110000</td>\n",
       "      <td>39.280000</td>\n",
       "      <td>188.500000</td>\n",
       "      <td>2501.000000</td>\n",
       "      <td>0.163400</td>\n",
       "      <td>0.345400</td>\n",
       "      <td>0.426800</td>\n",
       "      <td>0.201200</td>\n",
       "      <td>0.304000</td>\n",
       "      <td>...</td>\n",
       "      <td>49.540000</td>\n",
       "      <td>251.200000</td>\n",
       "      <td>4254.000000</td>\n",
       "      <td>0.222600</td>\n",
       "      <td>1.058000</td>\n",
       "      <td>1.252000</td>\n",
       "      <td>0.291000</td>\n",
       "      <td>0.663800</td>\n",
       "      <td>0.207500</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>8 rows × 32 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "                 id  radius_mean  texture_mean  perimeter_mean    area_mean  \\\n",
       "count  5.690000e+02   569.000000    569.000000      569.000000   569.000000   \n",
       "mean   3.037183e+07    14.127292     19.289649       91.969033   654.889104   \n",
       "std    1.250206e+08     3.524049      4.301036       24.298981   351.914129   \n",
       "min    8.670000e+03     6.981000      9.710000       43.790000   143.500000   \n",
       "25%    8.692180e+05    11.700000     16.170000       75.170000   420.300000   \n",
       "50%    9.060240e+05    13.370000     18.840000       86.240000   551.100000   \n",
       "75%    8.813129e+06    15.780000     21.800000      104.100000   782.700000   \n",
       "max    9.113205e+08    28.110000     39.280000      188.500000  2501.000000   \n",
       "\n",
       "       smoothness_mean  compactness_mean  concavity_mean  concave points_mean  \\\n",
       "count       569.000000        569.000000      569.000000           569.000000   \n",
       "mean          0.096360          0.104341        0.088799             0.048919   \n",
       "std           0.014064          0.052813        0.079720             0.038803   \n",
       "min           0.052630          0.019380        0.000000             0.000000   \n",
       "25%           0.086370          0.064920        0.029560             0.020310   \n",
       "50%           0.095870          0.092630        0.061540             0.033500   \n",
       "75%           0.105300          0.130400        0.130700             0.074000   \n",
       "max           0.163400          0.345400        0.426800             0.201200   \n",
       "\n",
       "       symmetry_mean     ...       texture_worst  perimeter_worst  \\\n",
       "count     569.000000     ...          569.000000       569.000000   \n",
       "mean        0.181162     ...           25.677223       107.261213   \n",
       "std         0.027414     ...            6.146258        33.602542   \n",
       "min         0.106000     ...           12.020000        50.410000   \n",
       "25%         0.161900     ...           21.080000        84.110000   \n",
       "50%         0.179200     ...           25.410000        97.660000   \n",
       "75%         0.195700     ...           29.720000       125.400000   \n",
       "max         0.304000     ...           49.540000       251.200000   \n",
       "\n",
       "        area_worst  smoothness_worst  compactness_worst  concavity_worst  \\\n",
       "count   569.000000        569.000000         569.000000       569.000000   \n",
       "mean    880.583128          0.132369           0.254265         0.272188   \n",
       "std     569.356993          0.022832           0.157336         0.208624   \n",
       "min     185.200000          0.071170           0.027290         0.000000   \n",
       "25%     515.300000          0.116600           0.147200         0.114500   \n",
       "50%     686.500000          0.131300           0.211900         0.226700   \n",
       "75%    1084.000000          0.146000           0.339100         0.382900   \n",
       "max    4254.000000          0.222600           1.058000         1.252000   \n",
       "\n",
       "       concave points_worst  symmetry_worst  fractal_dimension_worst  \\\n",
       "count            569.000000      569.000000               569.000000   \n",
       "mean               0.114606        0.290076                 0.083946   \n",
       "std                0.065732        0.061867                 0.018061   \n",
       "min                0.000000        0.156500                 0.055040   \n",
       "25%                0.064930        0.250400                 0.071460   \n",
       "50%                0.099930        0.282200                 0.080040   \n",
       "75%                0.161400        0.317900                 0.092080   \n",
       "max                0.291000        0.663800                 0.207500   \n",
       "\n",
       "       Unnamed: 32  \n",
       "count          0.0  \n",
       "mean           NaN  \n",
       "std            NaN  \n",
       "min            NaN  \n",
       "25%            NaN  \n",
       "50%            NaN  \n",
       "75%            NaN  \n",
       "max            NaN  \n",
       "\n",
       "[8 rows x 32 columns]"
      ]
     },
     "execution_count": 4,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "data.describe()"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "dd03fdb7-3144-4150-92de-a3f2132f233d",
    "_uuid": "24c05ec20cf1358ff6fda1a37e338f020017caf8"
   },
   "outputs": [],
   "source": [
    "## Data visualisation and pre-processing\n",
    "\n",
    "First thing to do is to enumerate the diagnosis column such that M = 1, B = 0. Then, I set the ID column to be the index of the dataframe. Afterall, the ID column will not be used for machine learning"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {
    "_cell_guid": "8ab2f1c6-b174-4a0a-bf9f-7c94326b3e61",
    "_execution_state": "idle",
    "_uuid": "7bea9ce154dbe82c5d29588f2e27f00bfe3fdf1d",
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "data['diagnosis'] = data['diagnosis'].apply(lambda x: '1' if x == 'M' else '0')\n",
    "data = data.set_index('id')\n",
    "del data['Unnamed: 32']"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "14412b26-d247-41ee-880f-f361a88e4110",
    "_uuid": "81ac236c6c3c0e8206956433426a02779479c094"
   },
   "outputs": [],
   "source": [
    "Let's take a look at the number of Benign and Maglinant cases from the dataset. From the output shown below, majority of the cases are benign (0)."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {
    "_cell_guid": "f5936456-74bf-4c42-824f-1973566313c3",
    "_execution_state": "idle",
    "_uuid": "1ec0c60db5f383231016f03dd8e8ff362b08dbc6"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "diagnosis\n",
      "0    357\n",
      "1    212\n",
      "dtype: int64\n"
     ]
    }
   ],
   "source": [
    "print(data.groupby('diagnosis').size())"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "3c027c06-7285-4bd4-b2c4-ad21e1e293da",
    "_uuid": "6baf17aab6d5183b80e7f3a9aadaca3dded6c3b6"
   },
   "outputs": [],
   "source": [
    "Next, we visualise the data using density plots to get a sense of the data distribution. From the outputs below, you can see the data shows a general gaussian distribution. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {
    "_cell_guid": "b5457977-bd86-4a6f-87da-8d1acc361852",
    "_execution_state": "idle",
    "_uuid": "40e65d54d983261df308a4aa138b649e4b86725a"
   },
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f2df0fce588>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "data.plot(kind='density', subplots=True, layout=(5,7), sharex=False, legend=False, fontsize=1)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "8b3bc827-7651-4be4-800a-e16727af83bc",
    "_uuid": "61fd93395d08de8251226f954320b8cbf4c02871"
   },
   "outputs": [],
   "source": [
    "It is good to check the correlations between the attributes. From the output graph below, The red around\n",
    "the diagonal suggests that attributes are correlated with each other. The yellow and green patches suggest some moderate correlation and the blue boxes show negative correlations. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {
    "_cell_guid": "7025c069-528a-4f68-902d-6964ccbb2eb5",
    "_execution_state": "idle",
    "_uuid": "30721fd7c6e1b4bc1b4c711db44a094969a92b1e"
   },
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f2decd376a0>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "from matplotlib import cm as cm\n",
    "\n",
    "fig = plt.figure()\n",
    "ax1 = fig.add_subplot(111)\n",
    "cmap = cm.get_cmap('jet', 30)\n",
    "cax = ax1.imshow(data.corr(), interpolation=\"none\", cmap=cmap)\n",
    "ax1.grid(True)\n",
    "plt.title('Breast Cancer Attributes Correlation')\n",
    "# Add colorbar, make sure to specify tick locations to match desired ticklabels\n",
    "fig.colorbar(cax, ticks=[.75,.8,.85,.90,.95,1])\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "80f7e0c1-4e55-4803-bf95-e16e41994a07",
    "_uuid": "dacd3192e8f4b70e41d714beea8c537a1d5faac6"
   },
   "outputs": [],
   "source": [
    "Finally, we'll split the data into predictor variables and target variable, following by breaking them into train and test sets. We will use 20% of the data as test set."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {
    "_cell_guid": "e5f9262f-40c7-4935-9e72-f9be15e113e1",
    "_uuid": "770c88e9d8f8f5f0744046d593942f6d7efe1ec5",
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "Y = data['diagnosis'].values\n",
    "X = data.drop('diagnosis', axis=1).values\n",
    "\n",
    "X_train, X_test, Y_train, Y_test = train_test_split (X, Y, test_size = 0.20, random_state=21)"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "6eb9751e-a4e4-457e-b0e3-728334b80999",
    "_uuid": "e85ea911c9057cda526ed82e742c1fa3daff5746"
   },
   "outputs": [],
   "source": [
    "## Baseline algorithm checking\n",
    "\n",
    "From the dataset, we will analysis and build a model to predict if a given set of symptoms lead to breast cancer. This is a binary classification problem, and a few algorithms are appropriate for use. Since we do not know which one will perform the best at the point, we will do a quick test on the few appropriate algorithms with default setting to get an early indication of how each of them perform. We will use 10 fold cross validation for each testing.\n",
    "\n",
    "The following non-linear algorithms will be used, namely: **Classification and Regression Trees (CART)**, **Linear Support Vector Machines (SVM)**, **Gaussian Naive Bayes (NB)** and **k-Nearest Neighbors (KNN)**."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {
    "_cell_guid": "9d75c60c-5a84-4cf1-abc4-83ccf1a04ea5",
    "_uuid": "1b6ac44446c91123c1ceb2bd027e60ab5eb35b9c",
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "models_list = []\n",
    "models_list.append(('CART', DecisionTreeClassifier()))\n",
    "models_list.append(('SVM', SVC())) \n",
    "models_list.append(('NB', GaussianNB()))\n",
    "models_list.append(('KNN', KNeighborsClassifier()))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {
    "_cell_guid": "b3919d00-b7aa-4d62-b438-ee822ed6e50e",
    "_uuid": "971d250d87e73a7f30cc137e4625c30d8c5331c8"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "CART: 0.912029 (0.039630) (run time: 0.138211)\n",
      "SVM: 0.619614 (0.082882) (run time: 0.164310)\n",
      "NB: 0.940773 (0.033921) (run time: 0.019228)\n",
      "KNN: 0.927729 (0.055250) (run time: 0.027202)\n"
     ]
    }
   ],
   "source": [
    "num_folds = 10\n",
    "results = []\n",
    "names = []\n",
    "\n",
    "for name, model in models_list:\n",
    "    kfold = KFold(n_splits=num_folds, random_state=123)\n",
    "    start = time.time()\n",
    "    cv_results = cross_val_score(model, X_train, Y_train, cv=kfold, scoring='accuracy')\n",
    "    end = time.time()\n",
    "    results.append(cv_results)\n",
    "    names.append(name)\n",
    "    print( \"%s: %f (%f) (run time: %f)\" % (name, cv_results.mean(), cv_results.std(), end-start))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {
    "_cell_guid": "04fa9fd2-636c-4f94-94b8-1c9230c85a5a",
    "_uuid": "cc6a875d67a7d600f983e565332843fa98254f9e"
   },
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f2decc6b828>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig = plt.figure()\n",
    "fig.suptitle('Performance Comparison')\n",
    "ax = fig.add_subplot(111)\n",
    "plt.boxplot(results)\n",
    "ax.set_xticklabels(names)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "c5f330ec-666a-4ee6-85c8-bd06389005d9",
    "_uuid": "894f687f2f616f40217321a2690ff36d0789ca50"
   },
   "outputs": [],
   "source": [
    "From the initial run, it looks like GaussianNB, KNN and CART performed the best given the dataset (all above 92% mean accuracy). Support Vector Machine has a surprisingly bad performance here. However, if we standardise the input dataset, it's performance should improve.  "
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "71509d84-7641-4321-a7d2-60f77deb1e4a",
    "_uuid": "c2e22d7495f32f54e7a6f9971503b38a71a210f4"
   },
   "outputs": [],
   "source": [
    "## Evaluation of algorithm on Standardised Data\n",
    "\n",
    "The performance of the few machine learning algorithm could be improved if a standardised dataset is being used. The improvement is likely for all the models. I will use pipelines that standardize the data and build the model for each fold in the cross-validation test harness. That way we can get a fair estimation of how each model with standardized data might perform on unseen data."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {
    "_cell_guid": "d8b48313-697e-46cd-8a24-c862912e77cd",
    "_uuid": "2c785e3b14836708690209ac1249220ea98c2592"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "ScaledCART: 0.920966 (0.038259) (run time: 0.098808)\n",
      "ScaledSVM: 0.964879 (0.038621) (run time: 0.073377)\n",
      "ScaledNB: 0.931932 (0.038625) (run time: 0.027154)\n",
      "ScaledKNN: 0.958357 (0.038595) (run time: 0.040088)\n"
     ]
    }
   ],
   "source": [
    "import warnings\n",
    "\n",
    "# Standardize the dataset\n",
    "pipelines = []\n",
    "\n",
    "pipelines.append(('ScaledCART', Pipeline([('Scaler', StandardScaler()),('CART',\n",
    "                                                                        DecisionTreeClassifier())])))\n",
    "pipelines.append(('ScaledSVM', Pipeline([('Scaler', StandardScaler()),('SVM', SVC( ))])))\n",
    "pipelines.append(('ScaledNB', Pipeline([('Scaler', StandardScaler()),('NB',\n",
    "                                                                      GaussianNB())])))\n",
    "pipelines.append(('ScaledKNN', Pipeline([('Scaler', StandardScaler()),('KNN',\n",
    "                                                                       KNeighborsClassifier())])))\n",
    "results = []\n",
    "names = []\n",
    "with warnings.catch_warnings():\n",
    "    warnings.simplefilter(\"ignore\")\n",
    "    kfold = KFold(n_splits=num_folds, random_state=123)\n",
    "    for name, model in pipelines:\n",
    "        start = time.time()\n",
    "        cv_results = cross_val_score(model, X_train, Y_train, cv=kfold, scoring='accuracy')\n",
    "        end = time.time()\n",
    "        results.append(cv_results)\n",
    "        names.append(name)\n",
    "        print( \"%s: %f (%f) (run time: %f)\" % (name, cv_results.mean(), cv_results.std(), end-start))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {
    "_cell_guid": "5595bdce-4044-4a29-93ba-3003efdbc4fe",
    "_uuid": "09c717fbeeaf8622c7328733812f93f0f1183a43"
   },
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f2dec9e4048>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig = plt.figure()\n",
    "fig.suptitle('Performance Comparison')\n",
    "ax = fig.add_subplot(111)\n",
    "plt.boxplot(results)\n",
    "ax.set_xticklabels(names)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "10287c3c-59a8-492a-aead-13bef7ba71d1",
    "_uuid": "6f63b31910e59133a0d05d2fa88f93b14092cdd1"
   },
   "outputs": [],
   "source": [
    "Notice the drastic improvement of SVM after using scaled data. \n",
    "\n",
    "Next, we'll fine tune the performance of SVM by tuning the algorithm\n",
    "\n",
    "## Algorithm Tuning - Tuning SVM\n",
    "\n",
    "We will focus on SVM for the algorithm tuning. We can tune **two** key parameter of the SVM algorithm - the value of C and the type of kernel. The default C for SVM is 1.0 and the kernel is Radial Basis Function (RBF). We will use the grid search method using 10-fold cross-validation with a standardized copy of the sample training dataset. We will try over a combination of C values and the following kernel types 'linear', 'poly', 'rbf' and 'sigmoid"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {
    "_cell_guid": "57e17e0e-7eb1-493b-9e12-da601aeaca30",
    "_uuid": "68186b006294aa46d3cc6f9858445fc009c3e7c0"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Best: 0.969231 using {'C': 2.0, 'kernel': 'rbf'}\n",
      "0.964835 (0.026196) with: {'C': 0.1, 'kernel': 'linear'}\n",
      "0.826374 (0.058723) with: {'C': 0.1, 'kernel': 'poly'}\n",
      "0.940659 (0.038201) with: {'C': 0.1, 'kernel': 'rbf'}\n",
      "0.949451 (0.032769) with: {'C': 0.1, 'kernel': 'sigmoid'}\n",
      "0.962637 (0.029474) with: {'C': 0.3, 'kernel': 'linear'}\n",
      "0.868132 (0.051148) with: {'C': 0.3, 'kernel': 'poly'}\n",
      "0.958242 (0.031970) with: {'C': 0.3, 'kernel': 'rbf'}\n",
      "0.958242 (0.033368) with: {'C': 0.3, 'kernel': 'sigmoid'}\n",
      "0.956044 (0.030933) with: {'C': 0.5, 'kernel': 'linear'}\n",
      "0.881319 (0.050677) with: {'C': 0.5, 'kernel': 'poly'}\n",
      "0.964835 (0.029906) with: {'C': 0.5, 'kernel': 'rbf'}\n",
      "0.953846 (0.026785) with: {'C': 0.5, 'kernel': 'sigmoid'}\n",
      "0.953846 (0.031587) with: {'C': 0.7, 'kernel': 'linear'}\n",
      "0.885714 (0.038199) with: {'C': 0.7, 'kernel': 'poly'}\n",
      "0.967033 (0.037271) with: {'C': 0.7, 'kernel': 'rbf'}\n",
      "0.953846 (0.028513) with: {'C': 0.7, 'kernel': 'sigmoid'}\n",
      "0.951648 (0.028834) with: {'C': 0.9, 'kernel': 'linear'}\n",
      "0.887912 (0.038950) with: {'C': 0.9, 'kernel': 'poly'}\n",
      "0.967033 (0.037271) with: {'C': 0.9, 'kernel': 'rbf'}\n",
      "0.949451 (0.034009) with: {'C': 0.9, 'kernel': 'sigmoid'}\n",
      "0.953846 (0.026546) with: {'C': 1.0, 'kernel': 'linear'}\n",
      "0.890110 (0.038311) with: {'C': 1.0, 'kernel': 'poly'}\n",
      "0.967033 (0.033027) with: {'C': 1.0, 'kernel': 'rbf'}\n",
      "0.947253 (0.032755) with: {'C': 1.0, 'kernel': 'sigmoid'}\n",
      "0.956044 (0.025765) with: {'C': 1.3, 'kernel': 'linear'}\n",
      "0.894505 (0.039427) with: {'C': 1.3, 'kernel': 'poly'}\n",
      "0.967033 (0.028188) with: {'C': 1.3, 'kernel': 'rbf'}\n",
      "0.942857 (0.031144) with: {'C': 1.3, 'kernel': 'sigmoid'}\n",
      "0.958242 (0.024765) with: {'C': 1.5, 'kernel': 'linear'}\n",
      "0.896703 (0.039791) with: {'C': 1.5, 'kernel': 'poly'}\n",
      "0.967033 (0.028188) with: {'C': 1.5, 'kernel': 'rbf'}\n",
      "0.940659 (0.035237) with: {'C': 1.5, 'kernel': 'sigmoid'}\n",
      "0.956044 (0.021766) with: {'C': 1.7, 'kernel': 'linear'}\n",
      "0.903297 (0.033409) with: {'C': 1.7, 'kernel': 'poly'}\n",
      "0.967033 (0.024479) with: {'C': 1.7, 'kernel': 'rbf'}\n",
      "0.945055 (0.035539) with: {'C': 1.7, 'kernel': 'sigmoid'}\n",
      "0.956044 (0.021766) with: {'C': 2.0, 'kernel': 'linear'}\n",
      "0.909890 (0.033680) with: {'C': 2.0, 'kernel': 'poly'}\n",
      "0.969231 (0.022370) with: {'C': 2.0, 'kernel': 'rbf'}\n",
      "0.931868 (0.028237) with: {'C': 2.0, 'kernel': 'sigmoid'}\n"
     ]
    }
   ],
   "source": [
    "scaler = StandardScaler().fit(X_train)\n",
    "rescaledX = scaler.transform(X_train)\n",
    "c_values = [0.1, 0.3, 0.5, 0.7, 0.9, 1.0, 1.3, 1.5, 1.7, 2.0]\n",
    "kernel_values = ['linear', 'poly', 'rbf', 'sigmoid']\n",
    "param_grid = dict(C=c_values, kernel=kernel_values)\n",
    "model = SVC()\n",
    "kfold = KFold(n_splits=num_folds, random_state=21)\n",
    "grid = GridSearchCV(estimator=model, param_grid=param_grid, scoring='accuracy', cv=kfold)\n",
    "grid_result = grid.fit(rescaledX, Y_train)\n",
    "print(\"Best: %f using %s\" % (grid_result.best_score_, grid_result.best_params_))\n",
    "means = grid_result.cv_results_['mean_test_score']\n",
    "stds = grid_result.cv_results_['std_test_score']\n",
    "params = grid_result.cv_results_['params']\n",
    "for mean, stdev, param in zip(means, stds, params):\n",
    "    print(\"%f (%f) with: %r\" % (mean, stdev, param))"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "094e29a4-0b77-4578-995d-7aa600e4fcb8",
    "_uuid": "8a4634f59a3d171ef5d48a67d7961a7ffd8134ba"
   },
   "outputs": [],
   "source": [
    "We can see the most accurate configuration was SVM with an **RBF** kernel and **C=1.5**, with the accuracy of **96.92%**."
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "b59e3fd3-9b58-4ec9-8129-bd43dfbecd13",
    "_uuid": "8975c42f962cc0bde2f8ed8e763b021110e34b8f"
   },
   "outputs": [],
   "source": [
    "## Application of SVC on dataset\n",
    "\n",
    "Let's fit the SVM to the dataset and see how it performs given the test data."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {
    "_cell_guid": "d22dbf43-8e52-45be-90f1-e93239d624e2",
    "_uuid": "56cdb1896c4c52b05e655485efbc433a770cebd6"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Run Time: 0.004889\n"
     ]
    }
   ],
   "source": [
    "# prepare the model\n",
    "with warnings.catch_warnings():\n",
    "    warnings.simplefilter(\"ignore\")\n",
    "    scaler = StandardScaler().fit(X_train)\n",
    "X_train_scaled = scaler.transform(X_train)\n",
    "model = SVC(C=2.0, kernel='rbf')\n",
    "start = time.time()\n",
    "model.fit(X_train_scaled, Y_train)\n",
    "end = time.time()\n",
    "print( \"Run Time: %f\" % (end-start))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {
    "_cell_guid": "062dac6f-e9dd-45f3-bbce-ef55d921b43f",
    "_uuid": "7ee346d6b8f2a86c8b51b8237dceed178737ca4c",
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# estimate accuracy on test dataset\n",
    "with warnings.catch_warnings():\n",
    "    warnings.simplefilter(\"ignore\")\n",
    "    X_test_scaled = scaler.transform(X_test)\n",
    "predictions = model.predict(X_test_scaled)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {
    "_cell_guid": "334688fe-9900-4872-a0f9-ab2758964d89",
    "_uuid": "fafb1fdc0bd04148e85c9e069aff39c78e06ddc8"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy score 0.991228\n",
      "             precision    recall  f1-score   support\n",
      "\n",
      "          0       1.00      0.99      0.99        75\n",
      "          1       0.97      1.00      0.99        39\n",
      "\n",
      "avg / total       0.99      0.99      0.99       114\n",
      "\n"
     ]
    }
   ],
   "source": [
    "print(\"Accuracy score %f\" % accuracy_score(Y_test, predictions))\n",
    "print(classification_report(Y_test, predictions))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {
    "_cell_guid": "158e4ad0-e901-4769-bc61-b9cdc310813f",
    "_uuid": "73b0c7c8bf76432b2d98b8978e031397d0055c55"
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "[[74  1]\n",
      " [ 0 39]]\n"
     ]
    }
   ],
   "source": [
    "print(confusion_matrix(Y_test, predictions))"
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "93e258f1-8725-4f6b-9ee5-7b360cc77952",
    "_uuid": "e4d054eadfabe28710fe9eb5e49207c1ae2e6969"
   },
   "outputs": [],
   "source": [
    "We can see that we achieve an accuracy of 99.1% on the held-out test dataset. From the confusion matrix, there is only 1 case of mis-classification. The performance of this algorithm is expected to be high given the symptoms for breast cancer should exchibit certain clear patterns. "
   ]
  },
  {
   "cell_type": "markdown",
   "execution_count": null,
   "metadata": {
    "_cell_guid": "711380de-5eea-43ca-8aa0-0ea788c3593a",
    "_uuid": "21e532367c3d9ad464471ecdb51df70f839ee0b7"
   },
   "outputs": [],
   "source": [
    "## What else could be done\n",
    "\n",
    "1. Test the algorithm on KNN and GausianNB\n",
    "2. Test the data with Artificial Neural Net "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {
    "_cell_guid": "c6c26a28-063c-4efb-916f-81e5f1eaa202",
    "_uuid": "2c6792f4b693836e7af04656b0b9bb70c3384705",
    "collapsed": true
   },
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "coursera": {
   "course_slug": "python-machine-learning",
   "graded_item_id": "f9SY5",
   "launcher_item_id": "oxndk",
   "part_id": "mh1Vo"
  },
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
   "version": "3.6.1"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 1
}

