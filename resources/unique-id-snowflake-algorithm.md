## Snowflake for Generating the timebased sortable unique Id

The Snowflake algorithm, originally designed by Twitter, generates unique IDs in a highly distributed environment. It ensures that the IDs are sortable by time and unique across different services and machines. The algorithm generates a 64-bit unique ID, which can be divided into several parts:

1. **Timestamp**: A timestamp in milliseconds since a custom epoch.
2. **Data Center ID**: Identifies the data center.
3. **Worker ID**: Identifies the machine or instance.
4. **Sequence**: A counter that is incremented for IDs generated within the same millisecond.

### Snowflake ID Structure

```
+------------------------------------------------------------------+
| 1-bit (unused) | 41-bit Timestamp | 10-bit Data Center + Worker ID | 12-bit Sequence |
+------------------------------------------------------------------+
```

1. **1-bit (unused)**: Always set to 0.
2. **41-bit Timestamp**: Milliseconds since a custom epoch.
3. **10-bit Data Center + Worker ID**: 5-bit data center ID + 5-bit worker ID.
4. **12-bit Sequence**: Incremented counter within the same millisecond.

### Detailed Steps to Generate Snowflake ID

1. **Define constants**:
    - Custom epoch start.
    - Bit lengths for each part.
    - Maximum values for data center ID, worker ID, and sequence.

2. **Initialize variables**:
    - Data center ID and worker ID.
    - Last timestamp.
    - Sequence number.

3. **Generate the ID**:
    - Get the current timestamp.
    - If the current timestamp is the same as the last timestamp, increment the sequence.
    - If the sequence exceeds its maximum, wait for the next millisecond.
    - If the current timestamp is different from the last timestamp, reset the sequence to 0.
    - Update the last timestamp to the current timestamp.
    - Combine the timestamp, data center ID, worker ID, and sequence to form the 64-bit ID.

### Example Implementation in Java

Let's break down the Snowflake algorithm implementation line by line:

```java
public class SnowflakeIdGenerator {

    private final long epoch = 1609459200000L; // Custom epoch (Jan 1, 2021)
    private final long dataCenterIdBits = 5L;
    private final long workerIdBits = 5L;
    private final long sequenceBits = 12L;

    private final long maxDataCenterId = ~(-1L << dataCenterIdBits);
    private final long maxWorkerId = ~(-1L << workerIdBits);
    private final long maxSequence = ~(-1L << sequenceBits);

    private final long dataCenterIdShift = sequenceBits + workerIdBits;
    private final long timestampShift = sequenceBits + workerIdBits + dataCenterIdBits;
    private final long workerIdShift = sequenceBits;

    private final long dataCenterId;
    private final long workerId;
    private long lastTimestamp = -1L;
    private long sequence = 0L;

    public SnowflakeIdGenerator(long dataCenterId, long workerId) {
        if (dataCenterId > maxDataCenterId || dataCenterId < 0) {
            throw new IllegalArgumentException(String.format("Data center ID must be between 0 and %d", maxDataCenterId));
        }
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("Worker ID must be between 0 and %d", maxWorkerId));
        }
        this.dataCenterId = dataCenterId;
        this.workerId = workerId;
    }

    public synchronized long generateId() {
        long currentTimestamp = System.currentTimeMillis();

        if (currentTimestamp < lastTimestamp) {
            throw new RuntimeException("Clock moved backwards. Refusing to generate ID.");
        }

        if (currentTimestamp == lastTimestamp) {
            sequence = (sequence + 1) & maxSequence;
            if (sequence == 0) {
                currentTimestamp = waitUntilNextMillis(currentTimestamp);
            }
        } else {
            sequence = 0;
        }

        lastTimestamp = currentTimestamp;

        return ((currentTimestamp - epoch) << timestampShift)
                | (dataCenterId << dataCenterIdShift)
                | (workerId << workerIdShift)
                | sequence;
    }

    private long waitUntilNextMillis(long currentTimestamp) {
        while (currentTimestamp == lastTimestamp) {
            currentTimestamp = System.currentTimeMillis();
        }
        return currentTimestamp;
    }
}
```

### Explanation with Example

#### Example Setup

Assume:
- Custom epoch: January 1, 2021.
- Data center ID: 2.
- Worker ID: 3.
- Current timestamp: `1640995200000L` (represents a specific time after the custom epoch).
- Sequence number starts at 0.

1. **Define constants**:
    ```java
    private final long epoch = 1609459200000L; // Jan 1, 2021
    private final long dataCenterIdBits = 5L;
    private final long workerIdBits = 5L;
    private final long sequenceBits = 12L;
    ```

2. **Initialize variables**:
    ```java
    private final long dataCenterId = 2L;
    private final long workerId = 3L;
    private long lastTimestamp = -1L;
    private long sequence = 0L;
    ```

3. **Generate the ID**:
    - **Get the current timestamp**:
      ```java
      long currentTimestamp = System.currentTimeMillis(); // Assume 1640995200000L
      ```

    - **Check for clock going backwards**:
      ```java
      if (currentTimestamp < lastTimestamp) {
          throw new RuntimeException("Clock moved backwards. Refusing to generate ID.");
      }
      ```

    - **If the current timestamp is the same as the last timestamp, increment the sequence**:
      ```java
      if (currentTimestamp == lastTimestamp) {
          sequence = (sequence + 1) & maxSequence;
          if (sequence == 0) {
              currentTimestamp = waitUntilNextMillis(currentTimestamp);
          }
      } else {
          sequence = 0;
      }
      ```

    - **Update the last timestamp to the current timestamp**:
      ```java
      lastTimestamp = currentTimestamp;
      ```

    - **Combine the timestamp, data center ID, worker ID, and sequence**:
      ```java
      return ((currentTimestamp - epoch) << timestampShift)
              | (dataCenterId << dataCenterIdShift)
              | (workerId << workerIdShift)
              | sequence;
      ```

#### Example Calculation

1. **Timestamp Part**:
    - `currentTimestamp - epoch = 1640995200000L - 1609459200000L = 31536000000L`.
    - Shift left by 22 bits (timestampShift = 22):
      ```java
      (31536000000L << 22) = 132270479130828800000L
      ```

2. **Data Center ID Part**:
    - Shift left by 17 bits (dataCenterIdShift = 17):
      ```java
      (2L << 17) = 262144L
      ```

3. **Worker ID Part**:
    - Shift left by 12 bits (workerIdShift = 12):
      ```java
      (3L << 12) = 12288L
      ```

4. **Sequence Part**:
    - Sequence = 0 (for the first ID in the current millisecond).

5. **Combine All Parts**:
    ```java
    ((31536000000L << 22) | (2L << 17) | (3L << 12) | 0)
    = 132270479130828800000L | 262144L | 12288L | 0
    = 132270479131091040000L
    ```

The generated Snowflake ID is `132270479131091040000L`.

### Summary

The Snowflake algorithm combines the current timestamp, data center ID, worker ID, and sequence to generate a unique 64-bit ID. The timestamp ensures the IDs are time-sortable, while the data center ID and worker ID ensure uniqueness across different data centers and machines. The sequence counter prevents ID collisions within the same millisecond.