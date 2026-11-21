# SQL_CTE_LEARNING
Kumpulan lima tugas sql dari hackerrank menggunakan CTE 
---

## 1. The Report

**Soal:** Disuruh Untuk menampilkan `Name`, `Grade`, dan `Marks` pada siswa. Jika siswa dengan grade di bawah 8 namanya diganti menjadi `NULL`. Setelah itu urutkan berdasarkan grade DESC; untuk grade ≥ 8 urutkan nama ASC, sedangkan untuk grade < 8 urutkan berdasarkan marks ASC.

**Pendekatan:**
1. Gabungkan tabel `Students` dan `Grades` memakai `BETWEEN Min_Mark AND Max_Mark` untuk mendapatkan grade tiap siswa.
2. Gunakan `CASE WHEN` untuk menyembunyikan nama jika `Grade < 8`.
3. Urutkan dengan `Grade DESC, Name ASC, Marks ASC`. Karena nama bernilai `NULL` pada grade < 8, pengurutan otomatis jatuh ke `Marks`.

```sql
WITH grade_student AS(
    SELECT
     s.Name,
     s.Marks,
     g.Grade
    FROM Students s
    JOIN Grades g ON s.Marks BETWEEN g.Min_Mark AND Max_Mark 
)

SELECT
CASE WHEN Grade < 8 THEN NULL ELSE Name END AS Name,
Grade,
Marks
FROM grade_student
ORDER BY Grade DESC, Name ASC, Marks ASC;
```

---

## 2. Weather Observation Station 18

**Soal:** Disuruh untuk menghitung *Manhattan Distance* antara titik (lat terkecil, long terkecil) dan titik (lat terbesar, long terbesar) pada tabel `STATION`, dibulatkan 4 angka di belakang koma.

**Pendekatan:**
1. Ambil `MIN` dan `MAX` dari `LAT_N` dan `LONG_W` dalam satu CTE.
2. Melakukan penghitungan lalu gunakan `ABS`, lalu bulatkan dengan `ROUND(..., 4)`.

```sql
WITH weather_obs AS (
    SELECT
     MIN(lat_n)  AS a,
     MIN(long_w) AS b,
     MAX(lat_n)  AS c,
     MAX(long_w) AS d
    FROM station
)
SELECT ROUND(ABS(a - c) + ABS(b - d), 4)
FROM weather_obs;

```

---

## 3. Full Score

**Soal:** Disuruh untuk menampilkan `hacker_id` dan `name` hacker yang meraih nilai tertinggi pada lebih dari satu challenge. Urutkan berdasarkan jumlah nilai tertinggi terbanyak, lalu `hacker_id` naik.

**Pendekatan:**
1. `max_score_challenge`: ambil skor maksimum tiap challenge dari tabel `Difficulty` lewat `difficulty_level`.
2. `full_score_hacker`: cocokkan `Submissions` dengan skor maksimum tersebut, hitung per hacker, dan simpan yang jumlahnya `> 1` (`HAVING`).
3. `hacker_name`: gabungkan dengan `Hackers` untuk mendapatkan nama.
4. Urutkan `count_max_score DESC, hacker_id ASC`.

```sql
WITH max_score_challenge AS (
    SELECT
    c.challenge_id,
    d.score AS max_score
    FROM Challenges c
    JOIN Difficulty d ON c.difficulty_level = d.difficulty_level
),
full_score_hacker AS (
    SELECT
    s.hacker_id,
    COUNT(*) AS count_max_score
    FROM Submissions s
    JOIN max_score_challenge m ON s.challenge_id = m.challenge_id
    AND s.score = m.max_score
    GROUP BY s.hacker_id
    HAVING COUNT(*) > 1
),
hacker_name AS (
    SELECT
    f.hacker_id,
    h.name,
    f.count_max_score
    FROM full_score_hacker f
    JOIN Hackers h ON f.hacker_id = h.hacker_id
)
SELECT
hacker_id,
name
FROM hacker_name
ORDER BY count_max_score DESC, hacker_id ASC;

```

---

## 4. Ollivander's Inventory

**Soal:** Dari tongkat sihir yang tidak jahat (`is_evil = 0`), tampilkan `id`, `age`, `coins_needed`, dan `power` untuk tongkat dengan `coins_needed` paling murah di setiap kombinasi `age` dan `power`. Urutkan `power` menurun, lalu `age` menurun.

**Pendekatan:**
1. Gabungkan `Wands` dan `Wands_Property`, filter `is_evil = 0`.
2. Hitung `MIN(coins_needed)` per `(age, power)` dengan window function.
3. Simpan baris yang `coins_needed = coins_min`.

```sql
WITH oliv_inven AS(
    SELECT
    w.id,
    wp.age,
    w.coins_needed,
    w.power,
    MIN(coins_needed) OVER(PARTITION BY wp.age, w.power) AS coins_min
    FROM Wands w
    JOIN Wands_Property wp ON w.code = wp.code
    where wp.is_evil = 0
)

SELECT
id,
age,
coins_needed,
power
FROM oliv_inven
WHERE coins_needed = coins_min
ORDER BY power DESC, age DESC;
```

---

## 5. Contest Leaderboard

**Soal:** Disuruh untuk menampilkan `hacker_id`, `name`, dan total skor tiap hacker, di mana yang dihitung hanya **skor tertinggi per challenge**. Hacker dengan total skor 0 tidak ditampilkan. Urutkan berdasarkan total skor menurun.

**Pendekatan:**
1. `hacker_max`: ambil `MAX(score)` untuk setiap pasangan `(hacker_id, challenge_id)` agar submission berulang tidak dihitung ganda.
2. `nama_hacker`: gabungkan dengan `Hackers` untuk nama.
3. Jumlahkan skor maksimum per hacker, buang total 0 dengan `HAVING SUM(max_score) > 0`.

```sql
WITH hacker_max AS(
    SELECT
     hacker_id,
     challenge_id,
     MAX(score) as max_score
    FROM Submissions
    GROUP BY hacker_id, challenge_id
),
nama_hacker AS(
    SELECT
    hm.hacker_id,
    h.name,
    hm.max_score
    FROM hacker_max hm
    JOIN Hackers h ON hm.hacker_id = h.hacker_id
    
)

SELECT
hacker_id,
name,
SUM(max_score) AS score
FROM nama_hacker
GROUP BY hacker_id, name
HAVING SUM(max_score) > 0
ORDER BY score DESC;
```

---
