# LinkedinLearning_FirstSpringBootMicroservice

---
This repository is used to keep the note and source code of what I learnt from the course.

Course Link:
https://www.linkedin.com/learning/creating-your-first-spring-boot-microservice/using-the-exercise-files


## Chapter 1: Create a Spring Boot Microservice

You can deploy multiple instances from the same jar/war file in the same container/machine via command as below

`java -jar target/explorecali-0.0.1-SNAPSHOT.jar`
Default port is 8080

`java -Dserver.port=9090 -jar target/explorecali-0.0.1-SNAPSHOT.jar`

`java -Dserver.port=8888 -jar target/explorecali-0.0.1-SNAPSHOT.jar`

Quiz:

- The Spring Initializr website https://start.spring.io/ is the only way to create a Spring Boot Project.
  A: False

---

## Chapter 2: Leverage Spring Data JPA Repository Interfaces

Quiz:

- Which method looks up all the tours given a tour package code?
  A: List<Tour> findByTourPackageCode(String code)
- Given the following class declaration and the TourRatingRepository declaration that follows it, what should be substituted for T and ID?

```
@Entity
public class TourRating {
  @Id
  private String id;
  @Column
  private Integer score;
}

public interface TourRatingRepository extends CrudRepository<T, ID> {
}
```

    A:
    T = TourRating
    ID = String

- What is JPA an acronym for?
  A: Java Persistence API

---

## Chapter 3: Spring Data REST

HTTP GET /tourPackages
HTTP GET /tourPackages/<code>

### Create

HTTP POST /tourPackages {<request body>}

    TourPackage tp = new TourPackage();

      ...set attributes

    TourPackageRepository.save(TourPackage tp)

### READ

HTTP GET /tourPackages

    TourPackageRepository.findAll()

HTTP GET /tourPackages/<<code>>

    TourPackageRepository.findById(String code)

### UPDATE
```
HTTP PUT or PATCH /tourPackages/<code> {<request body>}

    TourPackage tp = TourPackageRepository.findById(String code)

        ...set attributes

    TourPackageRepository.save(TourPackage tp)
```
### DELETE
```
HTTP DELETE /tourPackages/<code>
TourPackageRepository.deleteById(String code)
```

### Search

Every Entity in Spring Data REST provides **search** API

HTTP GET

http://localhost:8080/tourPackages/search/findByName?name=<name>
http://localhost:8080/tourPackages/search/findByName?name=Backpack%20Cal

List<Tour> findByTourPackageCode(@Param("code") String code);

URL
`GET http://localhost:8080/tours/search/findByTourPackageCode?code=BC`


### Paging ans Sorting

- modify the Repositiry Interface from **extending CrudRepository** to **extending PagingAndSortingRepository**
- modify the return type of customized getFunction from **List<T>** to **Page<T>**
- Add the parameter of customized getFunction with **Pageable pageable**.

    ```
    public interface TourRepository extends PagingAndSortingRepository<Tour, Integer> {
        /**
         * Find Tours associated with the Tour Package.
         *
         * @param code tour package code
         * @return List of found tours.
         */
        Page<Tour> findByTourPackageCode(@Param("code")String code, Pageable pageable);
    }
    ```

URL of GET:
    `http://localhost:8080/tours?size=3&page=1&sort=title,asc`

URL of SEARCH:
    `http://localhost:8080/tours/search/findByTourPackageCode?code=CC&size=2&sort=title,desc`



### @RepositoryRestResource (exported = false)
Class Level Annotation
```
@RepositoryRestResource(collectionResourceRel = "packages", path = "packages")
public interface TourPackageRepository extends CrudRepository<TourPackage, String> {
```

### @RestReource (exported = false)
Method Level Annotation
    
```
@Override
@RestResource(exported = false)
<S extends TourPackage> S save(S entity);

...

@Override
@RestResource(exported = false)
void delete(TourPackage entity);
```

After setting above, tou will receive **405 Method Not Allowed** when you're trying to send POST/PUT/DELETE update command

### HAL Browser

use `GET http://localhost:8080`
and select  `GET http://localhost:8080/profile`
then select and see `http://localhost:8080/profile/packages`
You'll see the metadata of the REST APIs
We could use *HAL Browser* of Spring.Data to view the well-formated metadata

Add the library to pom.xml
```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-rest-hal-browser</artifactId>
</dependency>
```


### Quiz:

- How could you invoke this TourRepository query method from an API where Tour Package=Backpack Cali?
  `List<Tour> findByTourPackageCode(String code)`
  A: `GET http://localhost:8080/tours/search/findByTourPackageCode?code=BC`

- You only need to include the Spring Data REST dependency to expose Spring Data CrudRepositories as HTTP Endpoints.
  A: True

---

## Chapter 4: Expose RESTful APIs with Spring MVC

