# hse21_hw1

1) Создаем папку hw#1 и копируем туда исходные файлы с сервера по ссылке:
```
mkdir hw#1
cd hw#1
ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
2) Выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs с личным random seed:
```
seqtk sample -s712 oil_R1.fastq 5000000 > pe_R1.fastq
seqtk sample -s712 oil_R2.fastq 5000000 > pe_R2.fastq
seqtk sample -s712 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
seqtk sample -s712 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq
```
3) Удаление исходных .fastq файлов, так как они больше не нужны:
```
rm -r oil_R1.fastq
rm -r oil_R2.fastq
rm -r oilMP_S4_L001_R1_001.fastq
rm -r oilMP_S4_L001_R2_001.fastq
```
4) Оценка качества исходных чтений (fastQC) и получение по ним общей статистики (multiQC):
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```
Для скачивания файлов с сервера была использована программа WinSCP

5) С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры:
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
6) Cнова удаляем ненужные исходные файлы типа .fastq
```
rm pe_R1.fastq
rm pe_R2.fastq
rm mp_R1.fastq
rm mp_R2.fastq
```
7) Оценка качества подрезанных чтений и получение по ним общей статистики с помощью программы fastQC и multiQC :
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/
```
```
mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
```
```
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
До:
![image](https://user-images.githubusercontent.com/60548614/138936116-9dff7817-c184-4ad0-95c2-5185222936f1.png)
После (уменьшилась длина последовательностей, процент GC у paired-end подрос):
![image](https://user-images.githubusercontent.com/60548614/138936182-ccc8a9eb-8353-4a95-97ad-32584821eb76.png)

До:
![image](https://user-images.githubusercontent.com/60548614/138940836-d454f04d-04ce-49f3-8da1-c34b77a196e7.png)
После (улучшилось качество чтений):
![image](https://user-images.githubusercontent.com/60548614/138940874-d524bdc9-9763-40ba-881b-22099ef5c942.png)

До:
![image](https://user-images.githubusercontent.com/60548614/138941583-b2d82c88-b2fb-4000-99b3-b8e75187b521.png)
После (почти полностью были удалены адаптеры):
![image](https://user-images.githubusercontent.com/60548614/138941670-9ade8f28-5abc-408d-95a4-999e7e5c5421.png)


8) С помощью программы “platanus assemble” собираем контиги из подрезанных чтений:
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
9) Анализ полученных контигов (общее кол-во контигов, их общая длина, длина самого длинного контига, N50):
(реализовано в jupyter_notebook, который также приложен - ДЗ№1 Биоинформатика 3 курс Майнор)
![image](https://user-images.githubusercontent.com/60548614/138952434-426ac9e4-52d1-49e4-8fd3-0a304086d00a.png)


10) С помощью программы “ platanus scaffold” собрали скаффолды из контигов, а также из подрезанных чтений:

```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
11) Анализ полученных скаффолдов:
![image](https://user-images.githubusercontent.com/60548614/138952538-44b75902-8cc2-4e5b-8858-0fc87fca5c89.png)

12) Анализ самого длинного скафолда:
![image](https://user-images.githubusercontent.com/60548614/138956512-eacb6b0c-6ebd-478f-ae19-af28347d1ab8.png)

13) Выделим самый длинный скафолд в отдельный файл:
```
echo scaffold1_len3834058_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```

14) С помощью программы “ platanus gap_close” уменьшаем кол-во гэпов с помощью подрезанных чтений:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```

15) Опять вытаскиваем самый длинный скафолд: 
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
16) Анализ самого длинного скаффолда:

![image](https://user-images.githubusercontent.com/60548614/138960872-3d74fdb9-f0a3-40dd-98f9-62ea358aaa51.png)
