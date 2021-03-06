# Android Studio를 활용한 운동기록 애플리케이션의 제작
### 목차 :book:
1. [프로젝트 이름](#1-프로젝트-이름-memo)

2. [프로젝트 일정](#2-프로젝트-일정-calendar)

3. [기술 스택](#3-기술-스택-computer)

4. [Room 라이브러리를 활용한 데이터베이스의 구현](#4-room-라이브러리를-활용한-데이터베이스의-구현-floppy_disk)

5. [MPAndroidChart 라이브러리를 활용한 꺾은선그래프의 구현](#5-mpandroidchart-라이브러리를-활용한-꺾은선그래프의-구현-chart_with_upwards_trend)

6. [프로젝트 시연](#6-프로젝트-시연-iphone)

-----
### 1. 프로젝트 이름 :memo:
<img src="https://user-images.githubusercontent.com/61148914/104551372-1c8e0380-567a-11eb-829f-27914a2121b6.png" width="25%">

-----
### 2. 프로젝트 일정 :calendar:
* **2020.12.23 :** Android Studio 설치 및 기본 환경설정

* **2020.12.24 ~ 2020.12.27 :** 키, 몸무게에 따른 BMI 수치 및 몸상태 출력 기능 구현

* **2020.12.28 ~ 2020.12.30 :** Room 라이브러리를 활용한 데이터베이스의 구현

* **2020.12.31 ~ 2021.01.03 :** 날짜별 운동시간에 대한 데이터 처리 로직(삽입, 삭제, 수정) 구현

* **2021.01.04 ~ 2021.01.07 :** 설정 메뉴에 대한 UI 구성 및 기능 구현

* **2021.01.08 ~ 2021.01.13 :** MPAndroidChart 라이브러리를 활용한 그래프 구현 및 테스트

* **2021.01.14 ~ 2021.01.115 :** 그래프 출력에 대한 오류 수정

* **2021.01.16 :** 최종 테스트

* **2021.01.17 :** UI 보완 및 마무리

-----
### 3. 기술 스택 :computer:
* IDE
```
- Android Studio
```
* 언어
```
- Java
```
* 라이브러리
```
- Room
- MPAndroidChart
```

-----
### 4. Room 라이브러리를 활용한 데이터베이스의 구현 :floppy_disk:
**1) Room 이란?**   
ㆍ 안드로이드 앱에서 SQLite 데이터베이스를 쉽고 편리하게 사용할 수 있도록 하는 기능이다.  
ㆍ 사용자의 데이터를 로컬 데이터베이스에 저장하여 기기가 네트워크에 접근할 수 없을 때도 사용자가 콘텐츠를 탐색할 수 있다.    

**2) Room의 구성요소 (Database, Entity, Dao)**
```java
@Entity
public class UserData {
   
    @PrimaryKey
    private int date;
    private int time;


    public UserData(int date, int time) {
        this.date = date;
        this.time = time;
    }

    public int getDate() {
        return date;
    }

    public void setDate(int date) {
        this.date = date;
    }

    public int getTime() {
        return time;
    }

    public void setTime(int time) {
        this.time = time;
    }

    @Override
    public String toString() {
        return "UserData{" +
                "date=" + this.date +
                ", time=" + this.time +
                '}' + "\n";
    }
}
```
ㆍ Room을 구성하는 3가지 주요 요소 중 하나인 Enitity에 관한 클래스이다.   
ㆍ 데이터베이스에서 테이블 역할을 수행하며, 데이터베이스에 저장되는 데이터를 정의한 클래스이다.   
ㆍ "@Entity" 어노테이션을 클래스 상단에 작성함으로써, Entity 역할을 하는 클래스임을 명시한다.   
ㆍ "@PrimaryKey" 어노테이션으로 기본 키 값을 지정할 수 있으며, 기본 키는 중복될 수 없다.   
<br/>
```java
@Dao
public interface UserDataDao {
  
    @Query("SELECT * FROM UserData")
    List<UserData> getAll();

    @Query("DELETE FROM UserData")
    void deleteAll();

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void put(UserData userData);

    @Update
    void update(UserData userDate);

    @Delete
    void delete(UserData userData);
}
```
ㆍ Room을 구성하는 3가지 주요 요소 중 하나인 DAO에 관한 인터페이스이다.   
ㆍ 데이터베이스에 접근하여 수행할 작업을 메소드 형태로 정의하였다.   
<br/>
```java
@androidx.room.Database(entities = {UserData.class}, version = 1)
public abstract class Database extends RoomDatabase {
    public abstract UserDataDao userDataDao();
}
```
ㆍ Room을 구성하는 3가지 주요 요소 중 하나인 Database에 관한 인터페이스이다.   
ㆍ 데이터베이스를 새롭게 생성하거나 버전을 관리하기위한 클래스이다.   
ㆍ 해당 인터페이스는 RoomDatabase 클래스를 상속받는 추상 클래스여야 한다.   
ㆍ "@Database" 어노테이션을 통해 Database 역할을 하는 인터페이스임을 명시해야 하며, 어노테이션 안에 해당 데이터베이스와 관련된 Entity 리스트를 포함해야 한다.   

**3) 사용자의 데이터를 데이터베이스에 삽입**
```java
saveData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (TextUtils.isEmpty(putDate.getText()) || TextUtils.isEmpty(putTime.getText())) {
                    Toast.makeText(getApplicationContext(), R.string.informationMessage, Toast.LENGTH_SHORT).show();
                } else {
                    try {
                        db.userDataDao().put(new UserData(Date.getDate(putDate), Time.getTime(putTime)));
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                    Toast.makeText(getApplicationContext(), R.string.saveMessage, Toast.LENGTH_SHORT).show();

                    putDate.setText("");
                    putTime.setText("");
                }
            }
        });
```
ㆍ 위 코드는 "Register_Activity.java" 파일에서 saveData 버튼을 눌렀을 때 동작을 정의한 메소드이다.   
ㆍ 사용자의 운동일자와 운동시간에 관한 데이터를 Entity로써 정의한 "UserData" 객체 형태로 생성한다.   
ㆍ 그 후, DAO 인터페이스에 정의한 "put()" 메소드를 통해 데이터베이스에 데이터를 삽입한다.   

-----
### 5. MPAndroidChart 라이브러리를 활용한 꺾은선그래프의 구현 :chart_with_upwards_trend:
**1) MPAndroidChart 란?**   
ㆍ Philipp Jahoda가 개발한 애플리케이션을 위한 차트 라이브러리이다.   
ㆍ 다양한 종류의 그래프들과 그래프를 제어할 수 있는 다양한 이벤트들을 제공해주고 있다.   

**2) 라이브러리 추가**
```java
implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'
```
ㆍ app 수준의 build.gradle 파일의 dependencies 부분에 위 코드를 작성하여 MPAndroidChart 라이브러리를 추가해준다.   

**3) Chart 태그 추가**
```java
<com.github.mikephil.charting.charts.LineChart
        android:id="@+id/graph"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginBottom="8dp"
        app:layout_constraintBottom_toTopOf="@+id/graphMenuButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
```
ㆍ 자신이 사용하고 싶은 layout 파일에 Chart 태그를 작성해준다.   
ㆍ LineChart외에도 다양한 Chart를 사용할 수 있다.   

**4) 데이터의 생성**
```java
ArrayList<Entry> values = new ArrayList<>();   // lineDataSet에 담겨질 데이터를 ArrayList 형태로 선언

/** lineDataSet에 담겨질 사용자의 운동시간 정보를 "value"에 삽입 */
for (int i = 0; i < userDataList.size(); i++) {
   int time = userDataList.get(i).getTime();

   values.add(new Entry(i, time));
}

if (values.isEmpty())   // "values"에 담겨진 데이터가 없을 경우 확대 및 축소 불가능
   getGraph.setScaleEnabled(false);
else   // "values"에 담겨진 데이터가 있을 경우 확대 및 축소 가능
   getGraph.setScaleEnabled(true);
```
ㆍ 사용자의 운동시간 정보를 저장할 ArrayList 변수인 "values"를 선언한다.   
ㆍ 이때 ArrayList의 타입은 라이브러리에서 지정한 Entry 타입이어야 한다.   
ㆍ 그 후, 사용자의 운동시간 정보를 "values" 변수에 삽입함으로써 그래프로 출력할 데이터 리스트를 생성한다.   

**5) LineDataSet의 생성**
```java
LineDataSet lineDataSet = new LineDataSet(values, "Exercise Time");   // ArrayList인 "values"를 참조해 LineDataSet 선언
lineDataSet.setColor(ContextCompat.getColor(context, R.color.easypink));   // 사용자에게 출력될 차트의 색상 설정
lineDataSet.setCircleColor(ContextCompat.getColor(context, R.color.easypink));   // 사용자에게 출력될 차트의 구분점 테두리 색상 설정
lineDataSet.setCircleHoleColor(ContextCompat.getColor(context, R.color.easypink));   // 사용자에게 출력될 차트의 구분점 구멍 색상 설정
lineDataSet.setDrawFilled(true);
Drawable graphFill = ContextCompat.getDrawable(context, R.drawable.graph_filled);   // 사용자에게 출력될 차트의 색상이 그라데이션으로 출력되도록 설정
lineDataSet.setFillDrawable(graphFill);
```
ㆍ 사용자에게 출력될 정보가 담겨진 "values"를 참조해 "lineDataSet"을 선언한다.   
ㆍ LineDataSet 변수는 사용자에게 보여질 하나의 그래프에 대한 변수이다.   
ㆍ 그래프 선의 색상, 구분점의 색상 등 다양한 설정이 가능하다.   

**6) LineData의 생성**
```java
LineData lineData = new LineData();   // LineDataSet을 담는 그릇으로써 여러개의 LineDatSet이 삽입 가능
lineData.addDataSet(lineDataSet);   // "lineData"에 위에서 선언한 "lineDataSet" 삽입
lineData.setValueTextColor(ContextCompat.getColor(context, R.color.lightblack));   // 사용자에게 출력될 차트의 텍스트 색상 설정
lineData.setValueTextSize(9);   // 사용자에게 출력될 차트의 텍스트 사이즈 설정
```
ㆍ LineDataSet을 담는 그릇과 같은 역할을 하는 LineData 변수를 생성한다. 이 때 여러개의 LineDataSet을 삽입할 수 있다.   
ㆍ 그래프의 텍스트 색상, 사이즈 등 다양한 설정이 가능하다.

**7) x축의 설정**
```java
XAxis xAxis = getGraph.getXAxis();   // x축에 해당하는 변수인 "xAxis" 선언
xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);   // x축의 위치 설정
xAxis.setTextColor(ContextCompat.getColor(context, R.color.lightblack));   // x축에 표시될 텍스트 색상 설정
xAxis.setGridColor(ContextCompat.getColor(context, R.color.superlightblack));   // x축의 색상 설정
xAxis.setGranularityEnabled(true);   // x축 데이터들의 간격 설정
xAxis.setGranularity(1f);   // x축 데이터들 간의 간격을 1f로 설정
xAxis.setLabelCount(5);   // x축에 표시될 구간을 최대 5개로 설정
```
ㆍ x축에 해당하는 변수인 "xAxis"를 선언한다.   
ㆍ x축에 관련된 다양한 설정이 가능하다.   

**8) x축 데이터 재가공**
```java
final ArrayList<String> xLabelList = new ArrayList<String>();   // x축에 표시될 데이터들을 ArrayList 형태로 선언

/** 데이터베이스에 저장된 사용자의 운동 일자 정보를 위에서 선언한 xLabelList에 추가 */
for (int i = 0; i < userDataList.size(); i++) {
   SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyMMdd");
   Date date = null;
      
   try {
      date = simpleDateFormat.parse(Float.toString(userDataList.get(i).getDate()));
   } catch (ParseException e) {
      e.printStackTrace();
   }

   SimpleDateFormat newFormat = new SimpleDateFormat("MM/dd");
   String dateString = newFormat.format(date);

   xLabelList.add(dateString);
}

/** x축에 표시될 데이터(xLabelList)를 재가공하여 출력 */
xAxis.setValueFormatter(new ValueFormatter() {

   @Override
   public String getFormattedValue(float value) {
      if (xLabelList.size() == 1) {
         if (value < 0)
            return "";
         else if (value == 1)
            return "";
         else
            return xLabelList.get((int) value);
       } else
            return xLabelList.get((int) value);
   }
});
```
ㆍ x축에 사용자의 운동 날짜 정보를 출력하기 위해 재가공하는 코드이다.   
ㆍ x축에 표시될 데이터 리스트인 String 타입의 ArrayList 변수 "xLabelList"를 선언한다.   
ㆍ 데이터베이스에 저장된 운동 날짜 데이터를 SimpleDataFormat() 메소드를 이용하여 적절한 날짜 형태로 변환후 "xLabelList"에 추가한다.   
ㆍ 최종적으로 getFormattedValue() 메소드를 String 타입의 날짜 형태로 출력하도록 재정의 한다.   

**9) y축의 설정**
```java
YAxis yAxisLeft = getGraph.getAxisLeft();   // 좌측 y축에 해당하는 변수인 "yAxisLeft" 선언
yAxisLeft.setTextColor(ContextCompat.getColor(context, R.color.lightblack));   // y축에 표시될 텍스트 색상 설정
yAxisLeft.setGridColor(ContextCompat.getColor(context, R.color.lightblack));   // y축의 색상 설정
yAxisLeft.setAxisMinimum(0f);   // y축에 표시될 최소 범위 설정
yAxisLeft.setAxisMaximum(160f);   // y축에 표시될 최대 범위 설정
yAxisLeft.setLabelCount(4);

YAxis yAxisRight = getGraph.getAxisRight();   // 우측 y축에 해당하는 변수인 "yAxisRight" 선언
yAxisRight.setDrawLabels(false);   // 우측 y축에 대한 미사용 설정
yAxisRight.setDrawAxisLine(false);   // 우측 y축에 대한 미사용 설정
yAxisRight.setDrawGridLines(false);   // 우측 y축에 대한 미사용 설정
```
ㆍ 좌측 y축에 해당하는 변수인 "yAxisLeft"를 선언한 후 색상, 범위 등 다양한 설정을 한다.      
ㆍ 우측 y축은 사용하지 않을 예정이므로 미사용에 대한 설정을 한다.   
 
**10) Legend의 설정**
```java
Legend legend = getGraph.getLegend();   // 레전드에 해당하는 변수인 "legend" 선언
legend.setEnabled(false);   // 레전드에 대한 미사용 설정
```
ㆍ 그래프 하단에 색과 라벨을 나타내는 Legend에 대한 설정이다.   
ㆍ Legend는 사용하지 않을 예정이므로 미사용에 대한 설정을 한다.   

**11) Description의 설정**
```java
Description description = graph.getDescription();   // 디스크립션에 해당하는 변수인 "description" 선언
description.setEnabled(false);   // 디스크립션에 대한 미사용 설정
```
ㆍ 그래프 하단에 설명을 나타내는 Description에 대한 설정이다.   
ㆍ Description은 사용하지 않을 예정이므로 미사용에 대한 설정을 한다.   

**11) 그래프 삽입**
```java
getGraph.setData(lineData);   // 위에서 설정한 그래프를 최종적으로 삽입
```
ㆍ 위에서 설정한 그래프를 setData() 메소드를 사용하여 삽입을한다.   

-----
### 6. 프로젝트 시연 :iphone:
ㆍ 유튜브에서 확인하기 : [바로가기 링크](https://youtu.be/MMyog4W3ZDk)
<br/>
ㆍ 플레이스토어에서 확인하기 : [바로가기 링크](https://play.google.com/store/apps/details?id=com.Steadily.exerciseapplication)
