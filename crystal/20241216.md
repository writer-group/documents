# Springboot 3.x 와 Flyway를 쓰며 테스트/운영 DB 서버 밴더 다르게 사용하는 법

### 미리 결론

- 테스트와 운영 애플리케이션의 DB 종류를 다르게 쓰고 있어서 마이그레이션 쿼리문에 구문 오류 에러가 뜬다면

- 설정 파일에 placeholders 기능과 migration 폴더 세분화를 통해 DB별 쿼리 파일을 만들어 분기해주어 해결한다!

- 신기술은 ChatGPT에 물어보지 말고 동작원리를 최대한 이해하고, stackOverFlow나 공식문서를 확인하자!



---



### 문제상황

제가 하고 있는 프로젝트는 테스트와 운영에서 다른 종류의 DB를 사용하고 있습니다.

**테스트모드에선 Springboot가 자동으로 생성해주는 H2를 사용하고,**

**운영모드에선 MySql을 사용합니다.**

그러다보니, 마이그레이션에 사용되는 쿼리문이 특정 Database 사투리에 종속되어 다른 한 쪽에선 구문 오류가 발생했습니다.

처음에는 최대한 공용으로 쓸 수 있는 저수준의 쿼리문을 사용하다가, 도저히 이 묘수로는 안되는 상황이 벌어져 <u>DB 별로 알맞은 쿼리문을 따로 만들어 적용하기</u>를 시도했습니다.



---



### 내가 했던 시도들

1. 파일명을 달리 해서 함 (이때, 버전은 달리 해줘야 에러가 안남)

   Chat GPT가 알려주는 대로 그대로 해맑게 따라해봤는데 ... 실패!

   ```
   // h2 전용으로 만든 쿼리 파일이면 
   V2.1__add_time_column_to_members_h2.sql
   
   // mysql 전용으로 만든 쿼리 파일이면
   V2.2__add_time_column_to_members_mysql.sql
   ```

2. 폴더를 나눠서 시도 => 실패!

   ```
   src/resources/db/migration/h2
   src/resources/db/migration/mysql
   ```



3. test 하위에 resources 폴더 만들어서 시도 (spring) => 실패
   ![image-20241220013311779](https://raw.githubusercontent.com/S2uJeong/blogImages/main/images/image-20241220013311779.png)

   위의 공식문서를 참고하고 혹시나 되나 하고 시도해봤는데, 이 방법은 그저 테스트 data를 넣기 위한 방법이지, test db에 따로 쿼리 파일을 지정해주는 용도로 설명된 것이 아닙니다.

4. 파일 나누기 + placeholder 이용해서 yaml파일 분할해주기 => 성공

   참고 :

   https://stackoverflow.com/questions/51346712/setting-up-multiple-database-with-flyway

​

---



### 해결 과정

1. 소스 폴더를 아래와 같이 나눠 줍니다.

   ![image-20241220013311779](https://raw.githubusercontent.com/S2uJeong/blogImages/main/images/image-20241220013311779.png)

2. application 설정 파일에 각 모드(prod/test)에 따른 DB 밴더를 placeholders 기능을 사용해 지정해줍니다.

    - application-test.yaml

      ```yaml
      spring:
         flyway:
             locations: db/migration, db/specific/${spring.flyway.placeholders.dbType}
             placeholders:
            	   dbType: h2
      ```

    - application-prod.yaml

      ```yaml
      spring:
         flyway:
             locations: db/migration, db/specific/${spring.flyway.placeholders.dbType}
             placeholders:
            	   dbType: mysql
      ```



---



### 느낀점

평소에 ChatGPT를 보며 에러를 파악하고 해결방법을 찾기 위한 루트를 전체적으로 살펴보았습니다. 그 후, 세부적인 정보를 인터넷 서칭을 하고 적용해보고 해결하는 편입니다.

하지만 Flyway에 관한 질문은 ChatGPT가 올바른 답변을 하고 있지 않음을 많이 느꼈습니다. 아예 잘못된 걸 설명해주거나, 심각한 오류를 발생시킬 수 있는 방법을 알려주기도 했습니다.

예를 들어 저번 글에 제가 사례로 말씀드렸던 flyway_history table의 에러가 났던 sql에 해당하는 행을 삭제하는 방법이 있습니다. 이 방법을 chatGPT가 알려줬다가,, 수습하는데 시간이 더 들었다죠..

이유가 뭘까 고민해보았는데, 신기술의 경우 데이터가 아직 많이 모이지 않아 정확한 답변을 못 하는 것 같습니다.

신기술을 적용한 것에 대해 에러가 발생할 때는, 첫번째로는 그 기술의 주요 동작원리를 이해한다는 전제하에 접근해야 함을 느꼈습니다.

그렇다면 잘못된 정보를 보게 되더라도 대충은 이거 이상한데? 하고 감을 잡을 수 있을 것이기 때문입니다.

그리고 ChatGPT보다는 stackOverFlow같이 개발자들이 실시간으로 문제를 공유하는 커뮤니티나 공식문서를 확인하는 것이 좋은 방법일 것이라고 생각합니다.

