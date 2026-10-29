# Istanbul AVM-lərində Çoxölçülü Pərakəndə Analitika – note

## 1. Dataset

- **Mənbə:** Kaggle – [Customer Shopping Dataset](https://www.kaggle.com/datasets/mehmettahiraslan/customer-shopping-dataset), fayl: `customer_shopping_data.csv`. Başqa dataset istifadə olunmayıb.
- **Əhatə:** 99,457 əməliyyat, 8 kateqoriya, 10 AVM, 3 ödəniş üsulu. Tarix aralığı: 01.01.2021 – 08.03.2023.
- **Ümumi gəlir:** 68,551,366. Datasetdə valyuta göstərilməyib, ona görə məbləğlər "pul vahidi" kimi verilir.

## 2. Metodologiya

1. **Keyfiyyət yoxlaması.** `dim`, `glimpse`, boş dəyərlər (`is.na`), təkrar sətirlər və təkrar `invoice_no` (`duplicated`), `summary`, `table`. Nəticə: boş dəyər, təkrar və qeyri-adi dəyər yoxdur, ona görə datanı təmizləməyə ehtiyac olmayıb.
2. **Gəlirin hesablanması.** Hər kateqoriyada `price / quantity` sabit çıxdı (məs. Clothing 300.08, Technology 1,050.00). Deməli, `price` artıq alışın ümumi məbləğidir və **gəlir = `price`**. Yenidən `quantity`-yə vursaq, gəliri təxminən 3.7 dəfə şişirdərdik.
3. **Tarix.** `invoice_date` sütunu `as.Date(..., format = "%d/%m/%Y")` ilə tarixə çevrildi (format gün/ay/il). `lubridate` ilə `year`, `month` və `year_month` sütunları çıxarıldı.
4. **Ölçülər üzrə xərcləmə.** `category`, `shopping_mall`, `payment_method` və `gender` üzrə `group_by` + `summarise` ilə əməliyyat sayı, ümumi gəlir, orta alış məbləği və paylar (%) hesablandı. Kateqoriyalar və AVM-lər gəlirə görə sıralandı.
5. **Pivot cədvəl.** `pivot_wider`: sətir = AVM, sütun = kateqoriya, dəyər = ümumi gəlir. İki heatmap çəkildi (`geom_tile`): mütləq gəlir və hər AVM daxilində kateqoriya payı (%).
6. **İllik trend.** Kateqoriya üzrə 2021–2023 gəlir xətt qrafiki. **2023 natamamdır** (data 8 mart-a qədərdir), ona görə əlavə olaraq hər ilin yalnız yanvar–fevral ayları müqayisə edildi. Mövsümiliyi ölçmək üçün aylıq gəliri ayın gün sayına böldük (2021–2022).
7. **Cins müqayisəsi.** Qadın və kişi üçün ümumi xərc, orta alış məbləği və xərcləmənin kateqoriyalar üzrə bölgüsü (%). Qruplaşdırılmış bar chart (`geom_col(position = "dodge")`).
8. **Bonus.** MoM artımı = (bu ay − keçən ay) / keçən ay × 100, hər AVM üçün (`lag()`). Son ay (mart 2023) cəmi 8 gün olduğu üçün çıxarıldı. Hər kateqoriyada dominant ödəniş üsulu əməliyyat sayına görə tapıldı.
9. **Alətlər:** R, `tidyverse` (dplyr, tidyr, ggplot2), `lubridate`, `scales`, `knitr`.

## 3. AVM rəhbərliyi üçün 5 biznes insight

### 1. Gəlir üç yüksək qiymətli kateqoriyada cəmləşib

**Nə gördük:** Clothing, Shoes və Technology gəlirin **94.8%**-ni verir, halbuki əməliyyatların cəmi 49.8%-ni təşkil edir. Qalan beş kateqoriya (Cosmetics, Toys, Food & Beverage, Books, Souvenir) əməliyyatların 50.2%-i olsa da gəlirin cəmi **5.2%**-ni verir. Orta alış məbləği: Technology 3,157, Shoes 1,807, Clothing 901; Food & Beverage isə cəmi 15.67.

**Nə etməli:**
- Ən yaxşı yerləri və ən böyük sahələri Clothing, Shoes və Technology mağazalarına ayırmaq; yeni kirayəçi cəlb edərkən bu üç kateqoriyaya prioritet vermək.
- Ucuz kateqoriyalar gəlirə az təsir edir, amma əməliyyatların yarısını təşkil edir. Onları ziyarətçi gətirən kateqoriya kimi saxlamaq, qiymətləndirməni isə gəlirlə yox, ziyarətçi gətirmə effekti ilə aparmaq.

### 2. AVM-lərin gəlir fərqi əməliyyat sayından gəlir, orta alışdan yox

**Nə gördük:** Mall of Istanbul (20.2%), Kanyon (20.0%) və Metrocity (15.0%) birlikdə gəlirin **55.2%**-ni verir. Ən kiçik beş AVM-in hər biri təxminən 5%-dir (4.9–5.1%). AVM-in gəliri ilə əməliyyat sayı praktiki olaraq eyni hərəkət edir (korrelyasiya ≈ 1.00). Orta alış məbləği isə AVM-lər arasında çox yaxındır: 674.36 (Forum Istanbul) – 704.72 (Emaar Square Mall), fərq cəmi 4.5%. Kanyon gəlirdə 2-ci, orta alışda isə 4-cü yerdədir.

**Nə etməli:**
- Kiçik AVM-lər üçün əsas hədəf ziyarətçi sayı və alışa çevrilmədir (tədbirlər, kampaniyalar, nəqliyyat əlaqəsi). Qiymət və ya "premium" strategiyası əsaslı olmazdı, çünki orta alış hər yerdə demək olar ki, eynidir.
- Kiçik AVM-lərdə aylıq gəlir çox dalğalıdır (aylıq artımın standart sapması 18–35%, ilk üç AVM-də isə 9–11%). Bir aylıq düşməyə reaksiya verməzdən əvvəl 3–6 aylıq trendə baxmaq lazımdır.

### 3. Gəlir artmır, mövsümilik isə zəifdir

**Nə gördük:** 2021-dən 2022-ə ümumi gəlir cəmi **+0.18%** dəyişib (31.32 → 31.37 milyon). Ən böyük kateqoriya Clothing 2.05% azalıb (14.37 → 14.07 milyon), Shoes +1.65%, Technology +2.31%, Books +6.75%. 2023 natamamdır, amma eyni dövrü (yanvar–fevral) müqayisə etdikdə 2023 2022-dən **4.1%** yüksəkdir (ilkin siqnal, cəmi 2 ay). Gündəlik orta gəlir ən yüksək ay (iyul) ilə ən aşağı ay (noyabr) arasında cəmi 7.6% fərqlənir.

**Nə etməli:**
- Clothing-in azalması (təxminən 0.29 milyon) digər kateqoriyalardakı artımın demək olar ki, hamısını neytrallaşdırıb. Clothing mağazaları üçün kolleksiya və kampaniya tədbirləri planlaşdırmaq, yanvar–fevraldakı bərpanın davam edib-etmədiyini izləmək.
- Güclü mövsümilik olmadığı üçün kampaniya ilə süni zirvə yaratmaq imkanı var. Noyabr ən zəif aydır; onu festival və ya endirim kampaniyası üçün sınaq ayı seçib nəticəni əvvəlki illərin eyni ayı ilə müqayisə etmək olar.

### 4. Kişi və qadınlar eyni xərcləyir, fərq yalnız alıcıların sayındadır

**Nə gördük:** Qadınlar əməliyyatların **59.8%**-ni, kişilər 40.2%-ni təşkil edir. Bir alışda orta xərcləmə demək olar ki, eynidir: qadın 688.14, kişi 690.92. Xərcləmənin kateqoriyalara bölgüsü də eynidir (ən böyük fərq Shoes-da, cəmi 0.50 faiz bəndi; orta alış fərqi hər kateqoriyada 1%-dən azdır).

**Nə etməli:**
- Cinsə görə kateqoriya üzrə ayrıca təklif hazırlamağa ehtiyac yoxdur, çünki davranış eynidir.
- Kişilər bir alışda qadın qədər xərcləyir, amma alıcıların yalnız 40%-ni təşkil edir. Kişi auditoriyasına yönəlmiş kiçik test kampaniyaları (məs. Technology və Shoes üzrə) keçirib kişi alıcı sayının artıb-artmadığını yoxlamaq.

### 5. Nağd ödəniş hər kateqoriyada üstündür, hətta bahalı alışlarda

**Nə gördük:** Cash əməliyyatların **44.7%**-ni və gəlirin 44.8%-ni təşkil edir; ardınca Credit Card (35.1%) və Debit Card (20.2%) gəlir. Cash **8 kateqoriyanın hamısında** birincidir, payı 44.2% (Cosmetics) ilə 45.5% (Books) arasında dəyişir. Ən bahalı kateqoriya olan Technology-də (orta alış 3,157) belə alışların 44.7%-i nağd ödənilir.

**Nə etməli:**
- Ödəniş seçimi kateqoriyadan asılı olmadığı üçün AVM üçün vahid ödəniş strategiyası kifayətdir.
- Yüksək məbləğli kateqoriyalarda (Technology və Shoes, gəlirin 49.5%-i) bank tərəfdaşları ilə kart və ya taksit kampaniyaları keçirib nağd payını azaltmaq. Bu, kassalarda nağd pulla bağlı risk və xərcləri azaldır, kart vasitəsilə isə müştəri datası (loyallıq proqramı) toplamaq imkanı yaradır.

## 4. Məhdudiyyətlər

- 2023 natamamdır (8 mart-a qədər), ona görə illik müqayisədə 2023 ehtiyatla şərh olunmalıdır.
- Valyuta göstərilməyib və inflyasiya düzəlişi aparılmayıb. Gəlir mənfəət deyil (xərc və marja datada yoxdur).
- Ziyarətçi sayı datada yoxdur; əməliyyat sayı ziyarətçi sayının təxmini göstəricisi kimi istifadə olunub.
- Nəticələr müşahidədir, səbəb-nəticə əlaqəsini sübut etmir. Kiçik AVM-lərdə aylıq artım təsadüfi dalğalanmaya həssasdır.

## 5. Qrafiklə

- Pivot heatmap (AVM vs kateqoriya): "Pivot cədvəl" bölməsi
- Kateqoriya üzrə illik trend xətt qrafiki: "2021–2023-cü illər üzrə gəlir tendensiyası" bölməsi
- Cins və kateqoriya üzrə qruplaşdırılmış bar chart: "Kişi və qadın müştərilərin xərcləmə davranışları" bölməsi
- Bonus: MoM heatmap ("AVM-lər üzrə aylıq gəlir artımı") və ödəniş üsulu qrafiki ("Hər kateqoriyada ən çox istifadə edilən ödəniş üsulu")
