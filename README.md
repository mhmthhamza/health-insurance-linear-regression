# Sigorta Masrafı Modellemesi: Doğrusal Regresyon

Sağlık sigortası masraflarını **çoklu doğrusal regresyon** ile modelleyen; varsayım kontrolleri, model tanıları, etkileşim terimi ve model karşılaştırmasını içeren tekrarlanabilir bir R / R Markdown çalışması.

> Aynı verinin lojistik regresyon ile analizi: [health-insurance-logistic-regression](https://github.com/mhmthhamza/health-insurance-logistic-regression)

📄 **Rapor:** [`analysis.html`](analysis.html) (indirip tarayıcıda açın; kod katlanabilir, yan menüde içindekiler tablosu vardır)

## Soru

Yıllık sigorta masrafı (`charges`) yaş, BMI, çocuk sayısı, sigara kullanımı, cinsiyet ve bölge ile nasıl açıklanır ve bu modellemede regresyon varsayımları sağlanır mı?

## Veri

`data/insurance.csv`: 1338 gözlem, 7 değişken (yaş, cinsiyet, BMI, çocuk sayısı, sigara kullanımı, bölge, masraf). Eksik gözlem yoktur.

Kaynak: GitHub'daki `stedy/Machine-Learning-with-R-datasets` deposundaki kopya (*Machine Learning with R* kitabı ve Kaggle ile ilişkilendirilen açık veri seti). İlk kaynağı ve lisansı doğrulanmamıştır; **gerçek şirket verisi olduğuna dair belge yoktur, büyük olasılıkla simüle edilmiştir.** Bulgular gerçek bir sigorta portföyüne genellenemez.

## Yöntemler

| Aşama | Uygulanan teknikler |
|---|---|
| Hazırlık | Ön inceleme, kukla değişkenler (n−1), `set.seed(145)` ile %80/%20 ayırma |
| Doğrusal model | Çoklu regresyon (`lm`), log dönüşümü, **sigara × obezite etkileşimi** |
| Varsayımlar | Q-Q, Shapiro-Wilk, Lilliefors, Anderson-Darling, Breusch-Pagan, White (elle), Durbin-Watson, VIF |
| Sorun giderme | Adımsal seçim (forward/backward/both), Cook / standart artık / kaldıraç / DFFITS ile aykırı-etkili gözlem, ağırlıklı en küçük kareler (WLS), PCR |
| Performans | Test verisinde RMSE, MAE, R²; 10 katlı çapraz doğrulama |

## Başlıca bulgular

- Masrafı en çok **sigara** belirler; yaş, çocuk sayısı ve BMI da anlamlıdır. Cinsiyet ve bölge etkisi küçüktür.
- **Obezite yalnızca sigara içenlerde masrafı artırır** (sigara × obezite etkileşimi). Etkileşimle AIC ≈ 1199'dan ≈ 1090'a düşer, düzeltilmiş R² ≈ 0,81 olur, test MAE ve R² iyileşir.
- Çoklu bağlantı ve otokorelasyon yoktur. **Normallik ve sabit varyans varsayımları sağlanmaz;** log dönüşümü, adımsal seçim, etkileşim, aykırı gözlem çıkarma ve WLS bunu tam gidermez (etkileşim ve WLS varyans sorununu belirgin biçimde azaltır).
- 63 aykırı/etkili gözlemi çıkarmak test performansını kötüleştirir; bu gözlemler geçerlidir.
- Test ve çapraz doğrulamada ham ölçekli model RMSE'de en iyidir; MAE'de etkileşimli model en düşük ortalamaya sahiptir, ancak fark CV standart sapmaları içindedir.
- PCR gereksizdir (VIF ≈ 1; çapraz doğrulama tüm bileşenleri seçer).

## Çalıştırma

R (≥ 4.2) ve pandoc gerekir (RStudio ile birlikte gelir).

```r
install.packages(c("rmarkdown", "knitr", "car", "lmtest", "nortest", "pls",
                   "caret", "PerformanceAnalytics"))
rmarkdown::render("analysis.Rmd")   # analysis.html üretir
```

## Klasör yapısı

```
├── analysis.Rmd         # tüm analiz ve yorumlar (Türkçe)
├── analysis.html        # derlenmiş rapor
├── data/insurance.csv   # veri
└── README.md
```

## Sınırlılıklar

- Veri eğitim amaçlıdır; sonuçlar gerçek bir sigorta portföyüne genellenemez.
- Normallik ve sabit varyans varsayımları tam sağlanamamıştır; artık grafiklerindeki kalan kümelerin kaynağı belirlenememiştir.
- Obezite eşiği (BMI ≥ 30) dışarıdan alınmış bir kestirimdir; BMI sürekli bırakılıp etkileşim kurulduğunda sonuçlar değişebilir.
