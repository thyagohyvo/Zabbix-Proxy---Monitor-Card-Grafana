SELECT
    h.host                                          AS hostname,
    h.name                                          AS display_name,

    -- CPU utilization: system.cpu.util (float → history)
    (SELECT ROUND(hf.value, 2)
     FROM history hf
     WHERE hf.itemid = (
         SELECT itemid FROM items
         WHERE hostid = h.hostid AND key_ = 'system.cpu.util' AND status = 0
         LIMIT 1
     )
     ORDER BY hf.clock DESC LIMIT 1)               AS cpu_pct,

    -- Memory utilization: vm.memory.utilization (float → history)
    (SELECT ROUND(hf.value, 2)
     FROM history hf
     WHERE hf.itemid = (
         SELECT itemid FROM items
         WHERE hostid = h.hostid AND key_ = 'vm.memory.utilization' AND status = 0
         LIMIT 1
     )
     ORDER BY hf.clock DESC LIMIT 1)               AS ram_pct,

    -- Disk utilization: vfs.fs.size[/,pused] (float → history)
    (SELECT ROUND(hf.value, 2)
     FROM history hf
     WHERE hf.itemid = (
         SELECT itemid FROM items
         WHERE hostid = h.hostid AND key_ = 'vfs.fs.size[/,pused]' AND status = 0
         LIMIT 1
     )
     ORDER BY hf.clock DESC LIMIT 1)               AS disk_pct,

    -- Running processes: proc.num[,,run] (unsigned → history_uint)
    (SELECT hu.value
     FROM history_uint hu
     WHERE hu.itemid = (
         SELECT itemid FROM items
         WHERE hostid = h.hostid AND key_ = 'proc.num[,,run]' AND status = 0
         LIMIT 1
     )
     ORDER BY hu.clock DESC LIMIT 1)               AS proc_running,

    -- Agent ping: agent.ping (unsigned → history_uint)
    (SELECT hu.value
     FROM history_uint hu
     WHERE hu.itemid = (
         SELECT itemid FROM items
         WHERE hostid = h.hostid AND key_ = 'agent.ping' AND status = 0
         LIMIT 1
     )
     ORDER BY hu.clock DESC LIMIT 1)               AS agent_ping

FROM hosts h
WHERE
    h.status = 0
    AND h.host = 'NOME-DO-HOST'
