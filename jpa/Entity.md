### 2024. 10. 15 (화) 
### 우아하게 메타데이터 저장하기

데이터를 저장할때 '생성일자', '수정일자', '생성자', '수정자' 같은 메타 데이터는 유용하게 사용되며 또 자주 쓰인다.
Entity를 설계할때 매번 메타데이터를 삽입하기란 번거롭기 그지없다. 또한 데이터가 삽입될때 우리는 별도로 이런 메타데이터를 삽입하는 코드를 작성해야한다. 이런 문제들을 간단하게 해결할 수 있도록 JPA에서는 특별한 기능을 제공한다.

### CreatedDate, LastModifiedDate, CreatedBy, ModifiedBy

 - 생성일자, 수정일자, 생성자, 수정자를 자동으로 저장해주는 기능


### 사용방법
```
@Getter
@ToString
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class AuditingFields {

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    @CreatedDate
    @Column(nullable = false, updatable = false)
    protected LocalDateTime createdAt; // 생성일시

    @CreatedBy
    @Column(nullable = false, updatable = false, length = 100)
    protected String createdBy; // 생성자

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    @LastModifiedDate
    @Column(nullable = false)
    protected LocalDateTime modifiedAt; // 수정일시

    @LastModifiedBy
    @Column(nullable = false, length = 100)
    protected String modifiedBy; // 수정자
}


```

이런 클래스는 여러 Entity에 공통적으로 쓰이다보니 Entity로 설계하는것이 아닌 추상 클래스로 선언하여 상속받아 사용한다.
해서 abstract class로 선언했다.

### MappedSupperclass
Entity class들이 추상 클래스를 상속할 경우 추상클래스 내에 필드들을 인식할 수 있도록 해주는 기능이다.


### EntityListener
Jpa Entity에서 이벤트가 발생할때마다 특정 로직을 실행시킬 수 있는 어노테이션

### AuditingEntityListener.class

```aidl

@Configurable
public class AuditingEntityListener {

	private @Nullable ObjectFactory<AuditingHandler> handler;

	/**
	 * Configures the {@link AuditingHandler} to be used to set the current auditor on the domain types touched.
	 *
	 * @param auditingHandler must not be {@literal null}.
	 */
	public void setAuditingHandler(ObjectFactory<AuditingHandler> auditingHandler) {

		Assert.notNull(auditingHandler, "AuditingHandler must not be null");
		this.handler = auditingHandler;
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * persist events.
	 *
	 * @param target
	 */
	@PrePersist
	public void touchForCreate(Object target) {

		Assert.notNull(target, "Entity must not be null");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markCreated(target);
			}
		}
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * update events.
	 *
	 * @param target
	 */
	@PreUpdate
	public void touchForUpdate(Object target) {

		Assert.notNull(target, "Entity must not be null");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markModified(target);
			}
		}
	}
}
```

@PrePersist: 해당 엔티티를 저장하기 전

@PreUpdate: 해당 엔티티를 업데이트 하기 전

@PostLoad: 해당 엔티티를 새로 불러오거나 refresh 한 이후

@PostPersist: 해당 엔티티를 저장한 이후

@PostUpdate: 해당 엔티티를 업데이트 한 이후

@PreRemove: 해당 엔티티를 삭제하기 전

@PostRemove: 해당 엔티티를 삭제한 이후


