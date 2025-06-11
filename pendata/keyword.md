```{contents}
```

# Statistics & Mathematics

<!-- ## Mean, Median, Mode
## Variance, Standard Deviation

## Normalization
## Euclidean Distance
## Manhattan Distance
## Minkowski Distance

## K-Nearest Neighbors

## Local Outlier Factor
### Normalization
### StandardScaler
### K-Distance
### Euclidean Distance
### Reachability Distance
### Local Reachability Density
### Local Outlier Factor

## Naïve Bayes
### Bayes Theorem
### Posterior Probability
### Conditional Probability
### Prior Probability
### Prediction
### Gaussian Naïve Bayes
### Likelihood

## Linear Algebra (Vektor, Matriks)
## Kalkulus (Turunan, Gradien)

## Supervised vs Unsupervised Learning

## Regresi (Linear, Polynomial, Logistic)
## Klasifikasi (Decision Tree, SVM, k-NN)
## Clustering (k-Means, Hierarchical)
## Ensemble Learning (Random Forest, XGBoost)
## Neural Networks (Dasar-Dasar CNN/RNN)
## Reinforcement Learning (Konsep Dasar)

## RMSE, MAE (untuk regresi)
## Silhouette Score (untuk clustering) -->

### 2. **Markdown Editor dengan Mermaid Support**
Lorem:
- [Mermaid.live](https://mermaid.live/)
- [StackEdit](https://stackedit.io/)
- [StackEdit](https://stackedit.io/)
- [Obsidian](https://obsidian.md/)
- [Typora](https://typora.io/)
- GitHub README (pastikan didukung di repo kamu)

Contoh dalam markdown:
````markdown
```mermaid
graph TD
    A[Data Mentah] --> B[Preprocessing]
    B --> C[Perhitungan Manual]
    B --> D[Model Sklearn]
    C --> E[Aturan Decision Tree]
    D --> E
    E --> F[Pengujian 2 Data Baru]
    C --> G[Akurasi Manual]
    D --> H[Akurasi Sklearn]
    G --> I[Perbandingan]
    H --> I

Contoh dalam markdown:
```mermaid
graph LR
    A[Data Asli] --> B[Transformasi Fitur]
    B --> C[Data Baru: Sepal+Petal Area]
    C --> D{Decision Tree Manual}
    C --> E{Decision Tree Sklearn}
    D --> F[Prediksi Manual]
    E --> G[Prediksi Sklearn]
    F --> H[Akurasi vs Label Asli]
    G --> H
