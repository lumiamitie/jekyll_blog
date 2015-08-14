---
layout: post
published: true
title: Geocoding with R
mathjax: false
featured: false
comments: false
headline: Making blogging easier for masses
categories: R
tags: jekyll
---

![cover-image](../../images/rocks-waves.jpg)

## ggmap::geocode �Լ��� �̿��� �����ڵ�

**ggmap** ��Ű������ **geocode**��� �Լ��� �ִ�

������ ���ڷ� �־��ָ� ������ �浵���� ��ȯ���ְ�, �ɼǿ� ���� �� ���� ������ �ҷ����⵵ �Ѵ�

�׷��� �� �������� �����Ǿ� �ֱ� �������� ������������ �Է��ؾ� �ϰ� ��ȯ�Ǵ� �� ���� �����ּҰ� ��ȯ�ȴ�


```r
library(ggmap)
```

```
## Loading required package: ggplot2
```

```r
geocode('Seoul')
```

```
## Information from URL : http://maps.googleapis.com/maps/api/geocode/json?address=Seoul&sensor=false
```

```
##       lon      lat
## 1 126.978 37.56654
```

```r
geocode('Seoul', output='latlona')
```

```
##       lon      lat            address
## 1 126.978 37.56654 seoul, south korea
```

```r
geocode('Seoul', output='more')
```

```
##       lon      lat     type     loctype            address   north   south
## 1 126.978 37.56654 locality approximate seoul, south korea 37.6956 37.4346
##       east     west postal_code     country administrative_area_level_2
## 1 127.1823 126.7968        <NA> south korea                        <NA>
##   administrative_area_level_1 locality street streetNo point_of_interest
## 1                        <NA>    seoul   <NA>       NA              <NA>
##   query
## 1 Seoul
```

�Ʒ��� ���� �ѱ۷� ������ �Է��ϸ� ������ ����.

`geocode('����')`

�ɼ��� �Ϻ� �����ϸ� �ѱ۵� ����� �� ���� ������ �ϰ� ���������� ���������� ����� ã�� ���ߴ�

�ѱ��������� �Է��ϼ� �ѱ��ּҸ� ��ȯ�������� ��� �ؾ��ұ�?


---

## Geocoding with Google API (+rvest)

�ϴ� ���⼭�� ������ �浵 �ڷḸ ������ ������ �Ѵ�


`http://maps.googleapis.com/maps/api/geocode/xml?sensor=false&language=ko&address='����'`

������������ �� url�� ������ xml �����͸� �� �� �ִ�

url �߰��� �ִ� xml�� json���� �ٲ��ָ� json ������ �ڷᵵ ���� �� �ִ�

---

�׷��� �Ʒ������� `rvest`��Ű���� �̿��� �� �ּ��� xml�ڷḦ �������� ���ϴ� �׸� �����غ����� �Ѵ�


```r
library(rvest)
url = "http://maps.googleapis.com/maps/api/geocode/xml?sensor=false&language=ko&address='����'"
seoul_xml = xml(url, encoding='UTF-8')
```

---

location �±׸� �˻��ϸ� ���� �׸� ����, �浵 �ڷᰡ �ִ� ���� �� �� �ִ� 


```r
seoul_xml %>%
  xml_node('location')
```

```
## <location>
##   <lat>37.5665350</lat>
##   <lng>126.9779692</lng>
## </location>
```

---

�Ʒ� ó�� �ڽ��׸��� �ؽ�Ʈ�� �Ѳ����� �޾ƿ͵� �ǰ�


```r
seoul_xml %>%
  xml_node('location') %>%
  xml_children() %>%
  xml_text()
```

```
##           lat           lng 
##  "37.5665350" "126.9779692"
```

---

�Ʒ�ó�� �ʿ��� �׸� ���� �����ؼ� �޾ƿ͵� �ȴ�


```r
seoul_xml %>%
  xml_node('lat') %>%
  xml_text()
```

```
## [1] "37.5665350"
```

---

�⺻���� ����� �˾����� ������ ���� ���ؼ� ����/�浵 �ڷḦ ������ ����

���⼭�� �켱 ���� ���� ����� �ۼ��� �Ŀ�

�� ���� ���� api�� �����ؼ� ����, �浵���� �޾ƿ���

�� �۾��� for ���� �̿��� �ݺ��ؼ� ���ϴ� ����� �������� �Ѵ�


```r
# ����,�浵���� �������� �ϴ� ������ ����̴�
loc_list = c('����', '����', '�λ�')

# ������� �����ϱ� ���� �� ����
geocode_result = c()
for(loc in loc_list){
  
  # �˻�� ������ url
  url = "http://maps.googleapis.com/maps/api/geocode/xml?sensor=false&language=ko&address='"
  
  # �˻�� ���Խ��Ѽ� ���� ������ url
  geocode_url = paste0(url, loc, "'")
  
  # url���� utf-8 ���ڵ����� xml�ڷḦ �����´�
  geocode_xml = xml(geocode_url, encoding='UTF-8')
  
  # �������� �����´�
  geocode_lat = geocode_xml %>%
    xml_node('lat') %>%
    xml_text()
  
  # �浵���� �����´�
  geocode_lon = geocode_xml %>%
    xml_node('lng') %>%
    xml_text()
  
  # ����, ����, �浵�� 1��¥�� ������ ���������� �����Ѵ�
  # �� �� ����,�浵���� ���ڷ� ��ȯ�Ѵ�
  geocode_data = data.frame(location = loc, 
                            lat = as.numeric(geocode_lat), 
                            lon = as.numeric(geocode_lon)
                            )
  # ���� ������� ����� ������Ʈ�� �������Ѽ� ���� �����Ѵ�
  geocode_result = rbind(geocode_result, geocode_data)
}
geocode_result
```

```
##   location      lat      lon
## 1     ���� 37.56654 126.9780
## 2     ���� 36.35041 127.3845
## 3     �λ� 35.17955 129.0756
```

