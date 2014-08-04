
## Chapter 3

MySQL 내부는 크게 MySQL 엔진과 스토리지 엔진으로 구분된다. 

### 1. 내부구조

#### 1.1 MySQL 엔진

MySQL 엔진은 클라이언트 접속, 쿼리 요청을 처리하는 **커넥션 핸들러**, **SQL 파서**, 권한등을 처리하는 **전처리기**, 쿼리 최적화를 위한 **옵티마이저**로 구성된다. 부가적으로, 성능 향상을 위해 InnoDB 버퍼풀이나, MyISAM 키 캐시 같은 보조저장소를 포함할 수 있다.

#### 1.2 Storage 엔진

실제 데이터를 읽고 쓰는 부분은 **스토리지 엔진** 이 담당한다. MySQL 엔진은 스토리지 엔진의 **핸들러 API** 를 통해 읽기/쓰기를 요청하게 된다. 아래는 핸들러 API 를 통해 얼마나 많은 요청이 있었는지를 확인하는 명령.

```
mysql> SHOW GLOBAL STATUS LIKE 'Handler%';

// Result
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Handler_commit             | 1     |
| Handler_delete             | 0     |
| Handler_discover           | 0     |
| Handler_prepare            | 0     |
| Handler_read_first         | 6     |
| Handler_read_key           | 1     |
| Handler_read_last          | 0     |
| Handler_read_next          | 0     |
| Handler_read_prev          | 0     |
| Handler_read_rnd           | 0     |
| Handler_read_rnd_next      | 1845  |
| Handler_rollback           | 0     |
| Handler_savepoint          | 0     |
| Handler_savepoint_rollback | 0     |
| Handler_update             | 0     |
| Handler_write              | 1768  |
+----------------------------+-------+
16 rows in set (0.00 sec)
```

### 2. 스레딩

#### 2.1 클라이언트 스레드

Foreground Thread, Connetion Thread, Client Thread. 모두 같은 말이다. 서버에 접속된 사용자 수 만큼 존재하며, 커넥션 연결이 종료되면 **Thread Cache** 에 들어가거나, 사라진다. 스레드 캐시는 `thread_cache_size` 파라미터를 통해서 조정할 수 있다. 

일반적으로 클라이언트 스레드는 데이터 버퍼나 캐시로부터 데이터를 가져오고, 데이터가 캐싱되어있지 않을 경우 직접 디스크나 인덱스로부터 읽어와 작업을 처리한다. 

스토리지 엔진의 종류에 따라 클라이언트 스레드가 하는 일이 다른데, **MyISAM 엔진** 을 사용하는 테이블의 경우, 디스크 쓰기 작업까지 클라이언트 스레드가 처리한다. 반면 **InnoDB 엔진** 을 사용하는 테이블은 데이터 버퍼와 캐시 쓰기 까지만 클라이언트 스레드가, 남은 디스크 기록 작업은 백그라운드 스레드가 처리한다.

#### 2.2 백그라운드 스레드



