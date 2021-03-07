# mysql_sys.x-latest_file_io-

SELECT 
    IF((`information_schema`.`processlist`.`ID` IS NULL),
        CONCAT(SUBSTRING_INDEX(`performance_schema`.`threads`.`NAME`,
                        '/',
                        -(1)),
                ':',
                `performance_schema`.`events_waits_history_long`.`THREAD_ID`),
        CONVERT( CONCAT(`information_schema`.`processlist`.`USER`,
                '@',
                `information_schema`.`processlist`.`HOST`,
                ':',
                `information_schema`.`processlist`.`ID`) USING UTF8MB4)) AS `thread`,
    `performance_schema`.`events_waits_history_long`.`OBJECT_NAME` AS `file`,
    `performance_schema`.`events_waits_history_long`.`TIMER_WAIT` AS `latency`,
    `performance_schema`.`events_waits_history_long`.`OPERATION` AS `operation`,
    `performance_schema`.`events_waits_history_long`.`NUMBER_OF_BYTES` AS `requested`
FROM
    ((`performance_schema`.`events_waits_history_long`
    JOIN `performance_schema`.`threads` ON ((`performance_schema`.`events_waits_history_long`.`THREAD_ID` = `performance_schema`.`threads`.`THREAD_ID`)))
    LEFT JOIN `information_schema`.`PROCESSLIST` ON ((`performance_schema`.`threads`.`PROCESSLIST_ID` = `information_schema`.`processlist`.`ID`)))
WHERE
    ((`performance_schema`.`events_waits_history_long`.`OBJECT_NAME` IS NOT NULL)
        AND (`performance_schema`.`events_waits_history_long`.`EVENT_NAME` LIKE 'wait/io/file/%'))
ORDER BY `performance_schema`.`events_waits_history_long`.`TIMER_START`
