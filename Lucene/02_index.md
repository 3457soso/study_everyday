# Lucene 기초 다지기

 **출처 :** [**실전비급 아파치 루씬 7: 엘라스틱서치 검색 엔진을 향한 첫걸음**](https://book.naver.com/bookdb/book_detail.nhn?bid=14134564)

#### 목차

1. [루씬의 이해](https://github.com/3457soso/TIL/blob/master/Lucene/01_Intro.md)
2. **텍스트 색인**
   - [**색인 이해하기**](#1-색인-이해하기)
   - [**IndexWriter**](#2-IndexWriter)
   - [**도큐먼트와 필드**](#3-도큐먼트와-필드)
   - [**다양한 데이터 타입 필드**](#4-다양한-데이터-타입-필드)
   - [**DocValues**](#5-DocValues)
3. [텍스트 분석](https://github.com/3457soso/TIL/blob/master/Lucene/03_analyze.md)
4. [텍스트 검색과 질의 방법](https://github.com/3457soso/TIL/blob/master/Lucene/04_query.md)
5. [루씬의 고급 검색](https://github.com/3457soso/TIL/blob/master/Lucene/05_core.md)
6. [루씬 동작 방식 이해하기](https://github.com/3457soso/TIL/blob/master/Lucene/06_inside.md)
7. [다양한 확장 기능](https://github.com/3457soso/TIL/blob/master/Lucene/07_extensions.md)



___

## 텍스트 색인

### 1. 색인 이해하기 

#### 1) 색인의 개념

- **등장 배경** : 수많은 정보 속에서 데이터를 찾는 데에는 많은 시간과 자원이 소모된다.

  - **도큐먼트** : 이에 검색 엔진은 빠르고 효과적인 검색을 위해 **색인**이라는 목차와 유사한 일종의 문서를 작성한다.
  - 사용자 질의와 색인을 대조해 수많은 도큐먼트에서 **일치하는 도큐먼트를 빠르게 찾는다**

- **색인한다** : 전체 데이터에서 유의미한 텀을 **추출**하고 **정제**한 후 **저장**하는 과정!

  - 작업의 결과물을 **색인**이라고 부른다.

#### 2) 색인 과정

1. **텍스트 획득** : 도큐먼트를 수집하고 저장

   - 이메일이나 웹 페이지, 기사, 메모 등과 같은 검색 대상에서 **데이터를 추출**
   - **크롤러** : 정보를 자동으로 수집하는 프로그램. 검색 엔진의 데이터 수집으로 많이 사용된다.
   - **피드** : 일정한 시간에 만들어지고, **한 번 출판되면 변경할 수 없다**
     - **RSS 피드** : 기사나 블로그 포스팅의 업데이트 내역을 정리한 XML 기반 콘텐츠 배급 포맷...
   - **변환** : `docx`, `PDF`, `HTML`, `XML` 같은 포맷의 파일을 별도의 변환 도구를 이용해 다른 도구로 변환!
   - **도큐먼트 데이터 저장** : 정제된 데이터 저장소에서의 데이터 획득 방식

   

2. **텍스트 변환** : 도큐먼트를 색인에 알맞게 변환

   - 텍스트를 효과적으로 검색할 수 있도록 수집 데이터를 변경할지, 재구성할지 결정해야 함!

   - 여러 포맷의 텍스트를 분석 가능한 단위로 쪼개고, 이를 색인해서 효과적으로 인식 가능한 텀으로 변환!

   - **인덱스 텀 (Index Term)** : 위 과정을 거쳐 변환된 용어

   - **종류**

     - **텍스트 파싱** : 검색 대상이 되는 텍스트를 검색 엔진에서 인식 가능하게 변환하는 것
     - **텍스트 분석** : 검색 엔진이 인식 가능한 문서의 텍스트를 파싱해, 검색에 최적으로 분석한 후 변경, 재구성

     

3. **색인 생성** : 빠른 검색에 최적인 자료 구조를 생성하고, 색인에 저장

   - 텀을 검색 가능한 특별한 자료 구조 (역색인)으로 구성!
   - 역색인은 텀을 키로, 그 텀이 포함된 도큐먼트 정보를 값으로 **쌍을 이뤄 구성** → 추상적으로 **맵** 형태
   - 추출된 단어가 전체 데이터에서 **어디에 위치하는지** 역색인으로 구성하는 과정이다

#### 3) 루씬에서 역색인을 만드는 과정

1. **도큐먼트 텍스트**
   - 색인을 하려면 루씬에서 인식 가능한 구조로 **데이터를 정제** 해야 한다.
   - 검색에서, 찾는 정보가 담긴 단위가 **도큐먼트**다.
     - 색인하길 원하는 데이터는 **모두 도큐먼트에** 담기며, 이렇게 담긴 텍스트가 루씬에 전달되면 **색인 시작**!
2. **토큰 스캔**
   - 도큐먼트는 필드라는 항목으로 구성됨! 도큐먼트를 나타내는 여러 구성 요소는 **필드로 매핑**
   - 도큐먼트의 필드 유형에 따라 필드 내용에서 텀이 되는 부분을 찾는 과정이 **토큰 스캔**
3. **토큰 분석**
   - 토큰 스캔 과정에서 필드 내용이 토큰으로 분리되면, **이를 분석하고 정제**해야 함
4. **해시 테이블**
   - 역색인을 구성할 때, 대상 도큐먼트에서 텀 별로 등장하는지를 함께 기록해 **검색 신뢰성을 더 높인다**
   - **등장 횟수** 를 해시 테이블에 저정한다
5. **퀵 정렬과 병합 정렬**
   - 해시 테이블에서 텀의 노출 수를 도큐먼트별로 구했다면, **검색에 용이하도록 정렬**
6. **델타 인코딩**
   - 검색 데이터의 크기는 대체로 매우 크다! → 실제 디스크에 저장하기 전에 **데이터 양을 줄이는 압축 과정** 필요!
   - 델타 인코딩 (Delta Encoding)은 서로 인접한 값 사이를 함께 기록해 **데이터를 압축하는 기법**
7. **디스크 쓰기**
   - 모든 과정을 거친 최종 결과물은 **디스크에 저장**된다.

<br>

___

### 2. IndexWriter

#### 1) IndexWriter 클래스

- **주요 메소드**

  - `addDocument()` : 도큐먼트 추가
  - `close()` : 열린 모든 자원을 닫고 쓰기 잠금 해제
  - `commit()` : 보류 중인 모든 변경 사항을 색인에 적용하고, 참조된 모든 색인 파일 **동기화**해 변경 사항 적용
    - 추가 및 삭제된 도큐먼트, 세그먼트 병합, 추가된 색인 등
  - `flush()` : 모든 인메모리 세그먼트를 메모리로 이동! 디렉토리에 저장하지만 커밋은 하지 않음
  - `forceMerge()` : 병합 (Merge) 정책을 사용해 세그먼트가 병합될 때까지 병합

#### 2) IndexWriterConfig 클래스

- **색인 생성 모드 설정**
  - `OpenMode.CREATE` : 새로운 색인을 생성하거나 기존 색인을 덮어씀
  - `OpenMode.APPEND` : 기존 색인을 열어 추가
  - `OpenMode.CREATE_OR_APPEND` : 기존 색인이 없으면 생성하고, 있으면 기존 색인을 연다
- **램 버퍼 크기 설정** : `IndexWriterConfig.setRAMBufferSizeMB()` 메소드로 RAM 버퍼 크기 지정
- **색인 커밋 크기 설정** : 색인 작업이 커밋되는 단위 설정

#### 3) Directory 클래스

- **램 기반 색인** : 모든 색인을 **메모리에** 임시 저장! `RAMDirectory`

  - **Pros** : 디스크에 아무것도 쓰지 않아 `FSDirectory` 보다 빠르다.
  - **Cons** : RAM은 휘발성 메모리이므로 **자바가 중단되면 색인도 함께 사라짐**

- **파일 시스템 색인** : RAM 기반 색인과 달리 **실제 파일 시스템에 색인 저장**

<br>

___

### 3. 도큐먼트와 필드

#### 1) 도큐먼트

- **도큐먼트** : 색인과 검색 과정에 사용되는 기본 단위로, 하나 이상의 필드로 구성된다.
  - 관계형 데이터베이스의 **행(Row)**이라고 할 수 있음!
- **도큐먼트 추가**
  1. 디렉토리 생성
  2. 사용할 분석기를 설정
  3. 색인 생성에 필요한 `IndexWriter`를 구성
  4. **도큐먼트 생성**
  5. 도큐먼트에 필드를 추가
  6. 생성된 도큐먼트를 **색인에 추가** 하면 끝!
- **도큐먼트 삭제**
  - `IndexWriter`의 `deleteDocuments()`가 도큐먼트 삭제를 담당
  - 해당 메소드를 호출해도 **바로 삭제되지 않음**
  - 버퍼 때문에, `flush()`가 수행될 때 대상 색인에서 **실제로 삭제가 이루어짐**
- **도큐먼트 수정**
  -  `updateDocument()`로 수정할 도큐먼트를 **찾고**, **삭제**한 후 **추가**
  - `flush()`가 실행되면 수정 대상 도큐먼트를 **실제로 삭제** 한다

#### 2) 필드

- 도큐먼트의 구성 요소로, 관계형 데이터베이스의 **컬럼(Column)**이라고 할 수 있음!

  - 색인 (Index) > 도큐먼트 (Document) > 필드 (Field) > 필드 값 (Value) > 텀 (Term)

- 각각의 필드는 **필드의 이름, 타입, 값** 으로 구성된다.

- **종류**

  1. 색인에 필드 값을 **저장** (`Stored Field`)
     - `TextField` : 전체 텍스트를 검색하는 데 사용되는 필드
     - `StringField` : String 타입이긴 한데, 원본 데이터에서 토큰을 색인하는 데 사용됨
     - `StoredField` : 요약 결과를 보관하는 필드로, **값만 저장**
  2. 범위 검색 쿼리를 위함 : `Stored Field`를 사용하면 해당 필드를 범위 검색에 사용할 수 없다
     - `IntPoint` : int 타입을 색인한다 (이 외에도 `Long`, `Float`, `Double` Point가 있음)
  3. 정렬과 분류를 위함 (`DocValues`)
     - `StoredDocValuesField` : 정렬과 분류를 위해 `byte[]` 배열로 색인!
     - `StoredSetDocValuesField` : `SortedSet<byte[]>`로 색인 ...

#### 3) 텀

- **텀**은 색인의 내부 단어를 의미하는 기본 단위! 텍스트 검색을 효과적으로 하고자 텀을 생성한다

<br>

___

### 4. 다양한 데이터 타입 필드

#### 1) String

- 가장 기본인 문자열 타입으로, 색인 시 주로 사용된다!

- `StringField` : 토큰화 과정이 없다! 단일 토큰으로 역색인 구조에 저장!

  - 아이디 값 조회처럼 **필드 값 전체가 일치** 해야 하는 경우에 사용

- `TextField` : 색인이나 검색 시에 **텍스트를 분석기를 통해 토큰화** 함!

  - **토크나이저 (Tokenizer)** 와 **토큰 필터 (Token Filter)** 를 거쳐 토큰을 생성한다

#### 2) 숫자 타입

- 색인 시 숫자 타입에는 `StoredField`를 사용한다.
- `int`, `float`, `long`, `double` 형이 있다!

#### 3) Date 타입

- Date는 `String`이나 `Numeric` 타입으로 저장해야 한다

<br>

___

### 5. DocValues

- 루씬은 전문 검색의 효율성을 이유로 색인을 **역색인 구조로 저장**

  - 전문 검색 시에는 역색인이 유리하지만, 숫자의 범위 검색이나 집계 (Aggregation)의 경우는 X!!!!!!

    → 이를 보완하기 위해 숫자 + 날짜 검색 + 집계에 `DocValues` 라는 독특한 구조 사용

#### 1) 루씬의 캐시

- JVM 힙 메모리와 `DocValues` 이 두 가지 방법으로 집계 검색을 한다.

- **JVM 힙 메모리** (행 기반 (Row-Based))

  - 루씬의 집계 방식 중 하나!
  - 역색인 구조로 필드를 `Stored Field`로 저장하는 키-값
  - **Pros** : 힙 메모리에서 동작하므로 빠르다
  - **Cons** : 많은 양의 데이터를 정렬하거나, 그룹핑 등을 할때는 **자원 소비, 데이터 로딩 시간 ↑**

  > Elasticsearch에는 **JVM 힙 메모리** 를 `fielddata`라고 부르는 특별한 구조로 사용!
  >
  > 하지만 **JVM 힙 메모리** 자원의 소모가 많기 때문에, 기본적으로 **비활성화**

#### 2) DocValues 등장 배경 및 특징

- **DocValues** (열 기반 (Column-Based))

- 메모리를 효율적으로 사용하기 위해 **JVM 힙 메모리**가 아닌 **운영체제의 파일 시스템 캐시를 사용**

  - 색인 시 디스크를, 검색 시 **시스템 캐시** 를 이용한다.
  - 역색인은 정렬이나 그룹핑에는 효율적이지 않고, 세그먼트 병합이 많을수록 I/O도 비례해 증가해 **처리 속도 ↓**

- `DocValues` 필드는 OS 상에서 루씬에게 할당된 메모리에 **직접 매핑** 되는 디스크 기반 데이터 구조!

- **특징**

  - 컬럼 기반의 구조를 채택해 힙 메모리 사용에 영향을 주지 않으면서 **힙 메모리를 사용하는 것과 같은 성능** 
  - 행 기반의 집계성 필드를 저장하는 데 **최적의 구조**
  - *루씬 내부에서 도큐먼트 ID에 기반한 값에 빠르게 접근 가능*
  - 세그먼트 단위로 생성되고, 검색에 사용되는 역색인과 마찬가지로 **불변**

- **장점**

  - 힙 메모리 사용량 감소 및 가비지 컬렉션 속도 향상
  - 대용량 세그먼트를 메모리로 다시 로드하는 데 발생하는 **지연 시간 단축**

  > Elasticsearch에서는 집계와 정렬을 빠르게 하기 위해 `doc_values` 값이 `true`로 활성화
  >
  > 검색 엔진 내부적으로는 색인 시 두 가지 방법 (열 기준, 행 기준) 모두로 데이터 저장!
  >
  > → 하나는 사용자 질의에 빠르게 검색하기 위핸 **역색인**, 다른 하나는 집계를 위한 **DocValues**




