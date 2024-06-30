# RefreshToken 주기적으로 삭제하기
RefreshToken의 만료기간(expiration)이 지난 토큰은 삭제하도록한다.   
이 기능을 매일 00시 00분에 실행하도록하자.

주기적으로 어떤 메소드를 실행하려면   
1. SpringApplication 파일에 @EnableScheduling 추가   
2. Task 파일에 @Scheduled추가하고 실행할 메서드와 파일들을 구현한다.
3. TIMESTAMP와 호환을 위해 RefreshToken의 expiration 필드를 Date로 해야한다.   

```java
@Component
public class ExpriationCleanUpTask {

    private final RefreshRepository refreshRepository;

    public ExpriationCleanUpTask(RefreshRepository refreshRepository) {
        this.refreshRepository = refreshRepository;
    }

    @Scheduled(cron = "0 0 0 * * *") // 매일 자정(00시 00분)에 실행
    public void cleanUpExpiredData(){
        refreshRepository.deleteExpiredTokens();
    }

}
```

```java
public interface RefreshRepository extends JpaRepository<RefreshToken, Long> {

    Boolean existsByRefresh(String refresh);

    @Transactional
    void deleteByRefresh(String refresh);

    @Transactional
    @Modifying
    @Query("DELETE FROM RefreshToken r WHERE r.expiration < CURRENT_TIMESTAMP")
    void deleteExpiredTokens();
}

```
실행 전   
![스케줄 실행 전](https://github.com/koreaioi/TIL/assets/147616203/8ca5f44c-bc1e-49e1-922f-3dfce8c61f81)   
실행 후    
![스케줄 실행 후](https://github.com/koreaioi/TIL/assets/147616203/8e82823a-692e-4055-a130-32411998f638)   


