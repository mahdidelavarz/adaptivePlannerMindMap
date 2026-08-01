# Java/JPA Reference Model

**Status:** proposal for backend-owner review. These files are not production implementation until integrated into `app/backend` and tested.

## `CanonicalClock.java`

```java
package com.adaptiveplanner.application.port;
import java.time.*;
public interface CanonicalClock {
    Instant now();
    LocalDate currentLocalDate(ZoneId zoneId);
}
```

## `IdGenerator.java`

```java
package com.adaptiveplanner.application.port;
import java.util.UUID;
public interface IdGenerator { UUID nextId(); }
```

## `BaseCanonicalEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.CreationSource;
import jakarta.persistence.*;
import java.time.Instant;
import java.util.UUID;

@MappedSuperclass
public abstract class BaseCanonicalEntity {
    @Id
    @Column(nullable=false, updatable=false)
    protected UUID id;

    @Column(name="user_id", nullable=false, updatable=false)
    protected UUID userId;

    @Column(name="created_at", nullable=false, updatable=false)
    protected Instant createdAt;

    @Column(name="updated_at", nullable=false)
    protected Instant updatedAt;

    @Enumerated(EnumType.STRING)
    @Column(nullable=false, updatable=false, length=30)
    protected CreationSource source;

    @Version
    @Column(nullable=false)
    protected long version;

    protected BaseCanonicalEntity() {}
}
```

## `GoalEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.*;
import jakarta.persistence.*;
import java.time.*;

@Entity
@Table(name="goals")
public class GoalEntity extends BaseCanonicalEntity {
    @Column(nullable=false, length=200)
    private String title;

    @Column(name="desired_outcome", nullable=false, columnDefinition="text")
    private String desiredOutcome;

    @Enumerated(EnumType.STRING)
    @Column(nullable=false, length=20)
    private GoalStatus status;

    @Column(name="target_date")
    private LocalDate targetDate;

    @Column(name="review_date")
    private LocalDate reviewDate;

    @Enumerated(EnumType.STRING)
    @Column(name="review_date_source", length=30)
    private ReviewDateSource reviewDateSource;

    @Column(name="last_continuation_decision_at")
    private Instant lastContinuationDecisionAt;

    @Column(name="terminal_at")
    private Instant terminalAt;

    protected GoalEntity() {}
}
```

## `ProjectEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.*;
import jakarta.persistence.*;
import java.time.*;
import java.util.UUID;

@Entity
@Table(name="projects")
public class ProjectEntity extends BaseCanonicalEntity {
    @Column(name="goal_id")
    private UUID goalId;

    @Column(nullable=false, length=200)
    private String title;

    @Column(name="completion_meaning", columnDefinition="text")
    private String completionMeaning;

    @Enumerated(EnumType.STRING)
    @Column(nullable=false, length=20)
    private ProjectStatus status;

    @Column(name="target_date")
    private LocalDate targetDate;

    @Column(name="review_date")
    private LocalDate reviewDate;

    @Enumerated(EnumType.STRING)
    @Column(name="review_date_source", length=30)
    private ReviewDateSource reviewDateSource;

    @Column(name="terminal_at")
    private Instant terminalAt;

    protected ProjectEntity() {}
}
```

## `TaskEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.*;
import jakarta.persistence.*;
import java.time.*;
import java.util.UUID;

@Entity
@Table(name="tasks")
public class TaskEntity extends BaseCanonicalEntity {
    @Column(name="goal_id") private UUID goalId;
    @Column(name="project_id") private UUID projectId;
    @Column(nullable=false, length=200) private String title;
    @Column(columnDefinition="text") private String description;
    @Enumerated(EnumType.STRING) @Column(nullable=false, length=20) private TaskStatus status;
    @Enumerated(EnumType.STRING) @Column(nullable=false, length=20) private TaskPlacement placement;
    @Column(name="planned_date") private LocalDate plannedDate;
    @Column(name="review_date") private LocalDate reviewDate;
    @Enumerated(EnumType.STRING) @Column(name="review_date_source", length=30) private ReviewDateSource reviewDateSource;
    private LocalDate deadline;
    @Column(name="is_protected", nullable=false) private boolean protectedItem;
    @Column(name="protection_reason_code", length=80) private String protectionReasonCode;
    @Column(name="terminal_at") private Instant terminalAt;
    protected TaskEntity() {}
}
```

## `RoutineEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.*;
import com.fasterxml.jackson.databind.JsonNode;
import jakarta.persistence.*;
import org.hibernate.annotations.JdbcTypeCode;
import org.hibernate.type.SqlTypes;
import java.time.*;
import java.util.UUID;

@Entity
@Table(name="routines")
public class RoutineEntity extends BaseCanonicalEntity {
    @Column(name="goal_id") private UUID goalId;
    @Column(name="project_id") private UUID projectId;
    @Column(nullable=false, length=200) private String title;
    @Column(columnDefinition="text") private String description;
    @Enumerated(EnumType.STRING) @Column(nullable=false, length=20) private RoutineStatus status;
    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name="recurrence_definition", nullable=false, columnDefinition="jsonb")
    private JsonNode recurrenceDefinition;
    @Column(name="recurrence_timezone", nullable=false, length=64) private String recurrenceTimezone;
    @Column(name="effective_from_local_date", nullable=false) private LocalDate effectiveFromLocalDate;
    @Column(name="effective_until_local_date") private LocalDate effectiveUntilLocalDate;
    @Column(name="continuation_of_routine_id") private UUID continuationOfRoutineId;
    @Column(name="stopped_at") private Instant stoppedAt;
    protected RoutineEntity() {}
}
```

## `RoutineOccurrenceEntity.java`

```java
package com.adaptiveplanner.domain.model;
import com.adaptiveplanner.domain.model.enums.*;
import jakarta.persistence.*;
import java.time.*;
import java.util.UUID;

@Entity
@Table(
    name="routine_occurrences",
    uniqueConstraints=@UniqueConstraint(
        name="uq_occurrences_routine_date",
        columnNames={"routine_id","scheduled_local_date"}
    )
)
public class RoutineOccurrenceEntity extends BaseCanonicalEntity {
    @Column(name="routine_id", nullable=false, updatable=false)
    private UUID routineId;

    @Column(name="scheduled_local_date", nullable=false, updatable=false)
    private LocalDate scheduledLocalDate;

    @Enumerated(EnumType.STRING)
    @Column(nullable=false, length=20)
    private RoutineOccurrenceStatus status;

    @Column(name="resolved_at")
    private Instant resolvedAt;

    protected RoutineOccurrenceEntity() {}
}
```

## Enums

```java
public enum CreationSource { MANUAL, AI_ASSISTED, SYSTEM_MIGRATED }
public enum ReviewDateSource { USER, SYSTEM_DEFAULT, MIGRATED_DEFAULT }
public enum GoalStatus { ACTIVE, ACHIEVED, ABANDONED }
public enum ProjectStatus { ACTIVE, COMPLETED, STOPPED }
public enum TaskStatus { ACTIVE, COMPLETED, DROPPED }
public enum TaskPlacement { SCHEDULED, BACKLOG }
public enum RoutineStatus { ACTIVE, STOPPED }
public enum RoutineOccurrenceStatus { PENDING, DONE, MISSED }
```

## Repository ports

```java
public interface GoalRepository {
    Optional<GoalEntity> findByIdAndUserId(UUID id, UUID userId);
    GoalEntity save(GoalEntity entity);
}

public interface ProjectRepository {
    Optional<ProjectEntity> findByIdAndUserId(UUID id, UUID userId);
    ProjectEntity save(ProjectEntity entity);
}

public interface TaskRepository {
    Optional<TaskEntity> findByIdAndUserId(UUID id, UUID userId);
    List<TaskEntity> findToday(UUID userId, LocalDate date);
    List<TaskEntity> findExecutionOverdue(UUID userId, LocalDate date);
    List<TaskEntity> findReviewDue(UUID userId, LocalDate date);
    List<TaskEntity> findActiveByProjectId(UUID userId, UUID projectId);
    TaskEntity save(TaskEntity entity);
}

public interface RoutineRepository {
    Optional<RoutineEntity> findByIdAndUserId(UUID id, UUID userId);
    List<RoutineEntity> findActiveByProjectId(UUID userId, UUID projectId);
    Optional<RoutineEntity> findDirectContinuation(UUID sourceRoutineId);
    RoutineEntity save(RoutineEntity entity);
}

public interface RoutineOccurrenceRepository {
    Optional<RoutineOccurrenceEntity> findByRoutineIdAndScheduledLocalDate(UUID routineId, LocalDate date);
    List<RoutineOccurrenceEntity> findForUserAndDate(UUID userId, LocalDate date);
    RoutineOccurrenceEntity save(RoutineOccurrenceEntity entity);
}
```

## Mapping rule

These are reference mappings, not permission for direct field mutation. Domain/application command handlers must own construction and lifecycle transitions. Broad `CascadeType.ALL` relations are intentionally absent.
