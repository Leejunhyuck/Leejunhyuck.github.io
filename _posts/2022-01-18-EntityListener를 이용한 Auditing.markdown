---
layout: post
title: EntityListener를 이용한 Auditing
date: 2021-01-18 10:30:23 +0900
category: Jpa
---
# EntityListener를 이용한 Auditing
* 테이블 정보가 변경되면, 해당 내용을 히스토리성으로 기록하는 테이블을 생성하고 만들어야 함. 해당 작업은 Jpa의 EntityListner을 사용하여 진행 할 것임.

* History 적재를 위한 요구서

* 데이터베이스 테이블의 중요데이터의 변경점 이력을 남기기

### 상세 설계

* 정보
  * 중요 테이블의 변경 사항을 저장하여 운영상의 이점을 위하여 기록용 테이블에 저장하려 함.
  
* 요구사항
  * 테이블의 History 테이블을 각각 생성 후,  기존 테이블에 생성, 변경시 History 테이블에 변경사항을 저장한다.

* 고려사항
  1. 기존의 방식대로 데이터 생성, 변경시점에서 추가로 History 테이블에 변경하게 되면  3가지 문제점이 있다.
  2. 생성, 변경시점에 History 테이블에 똑같이 코드를 추가해야한다.
  3. 커스텀이 필요한 경우, 휴먼 에러의 가능성이 있다.
  4. History 테이블의 저장하는 코드가 중복해서 발생한다.

* 결론   
   따라서 Entity의 변경시점에 Listener를 등록 후 사용하면 해당 내용을 한번에 설정 해 두면 자동으로 동작이 가능하다.
   또한, 추가 구현 사항이 있을 경우 메서드를 추가 하여 확장성도 좋다.

* 구현  
  해당 내용을 구현하기 위해서, 기존과 테이블 칼럼과 같은 History 테이블을 생성 후 Entity별 Listener을 설정 후 사용한다.

* 추가 고려사항
  * 실무에서 작성시간과 수정시간만 필요하고 작성자와 수정자 작성시간과 수정시간이 모두 필요한 경우로 나뉨 따라서 해당 클래스를 나누었음
  * 추상 클래스 작성시간과 수정시간 - 1
  * 추상 클래스 작성자와 수정자 - 2
  * 1번을 상속받은 2번
  * entity에 작성시간과 수정시간만 필요한 경우 1번만 상속, 모두 필요한 경우 2번 상속

```
@MappedSuperclass
@EntityListeners(AuditingEntityListener::class)
abstract class BaseEntity : BaseTimeEntity() {
    @CreatedBy
    @Column(name= "reg_id", updatable = false)
    private var regId: String? = ""

    @LastModifiedBy
    @Column(name = "updt_id")
    private var updtId : String? = ""
}

```

```
@MappedSuperclass
@EntityListeners(AuditingEntityListener::class)
abstract class BaseEntity : BaseTimeEntity() {
    @CreatedBy
    @Column(name= "reg_id", updatable = false)
    private var regId: String? = ""

    @LastModifiedBy
    @Column(name = "updt_id")
    private var updtId : String? = ""
}

```

* Auditing 적용을 위한 설정
```
@Configuration
@EnableJpaAuditing
class JpaAuditingConfig {
    @Bean
    fun auditorProvider(): AuditorAware<String?> {
        return AuditorAware<String?> { Optional.of(UserContext.userLocal.get().email) }
    }
}
```

* Listener class 생성
```
class StateListener {
    @PrePersist
    fun prePersist(state: State) {
        saveStateHistory(state)
    }

    @PreUpdate
    fun preUpdate(state: State) {
        saveStateHistory(state)
    }

    private fun saveStateHistory(state: State) {
        val stateHistoryDao: stateHistoryDao = BeanUtils.getBean(
            stateHistoryDao::class.java
        )

        var stateHistory = StateHistory(
          stateId = state.stateId
        )

        stateHistoryDao.save(stateHistory)
    }
}

```
* Listener에서는 @Autowired 또는 생성자 주입방식을 사용하지 못하여 해당 기느을 하는 Util이 필요함

```
@Component
class BeanUtils : ApplicationContextAware {
    @Throws(BeansException::class)
    override fun setApplicationContext(applicationContext: ApplicationContext) {
        Companion.applicationContext = applicationContext
    }

    companion object {
        private var applicationContext: ApplicationContext? = null
        fun <T> getBean(cls: Class<T>): T {
            return applicationContext!!.getBean(cls)
        }
    }
}
```