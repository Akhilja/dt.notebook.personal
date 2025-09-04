fetch dt.entity.process_group_instance
| filterOut isNull(logFileStatus)
| fieldsAdd sLogSourceState = toString(logFileStatus)
| fieldsAdd sLogSourceState = substring(sLogSourceState, from:1, to: stringLength(sLogSourceState)-1)
| fieldsAdd sLogSourceState = splitString(sLogSourceState,",")
| expand sLogSourceState
| fieldsAdd sLogSourceState=trim(sLogSourceState)
| parse sLogSourceState, """'"'LD:file_name '":"'LD: status'"'"""
| fieldsAdd host=belongs_to[dt.entity.host]
| fieldsAdd hostGroup=entityAttr(host, type:"dt.entity.host", "hostGroupName")
| fields id, file_name, status, hostGroup
| append [fetch dt.entity.host
| filterOut isNull(logFileStatus)
| fieldsAdd sLogSourceState = toString(logFileStatus)
| fieldsAdd sLogSourceState = substring(sLogSourceState, from:1, to: stringLength(sLogSourceState)-1)
| fieldsAdd sLogSourceState = splitString(sLogSourceState,",")
| expand sLogSourceState
| fieldsAdd sLogSourceState=trim(sLogSourceState)
| parse sLogSourceState, """'"'LD:file_name '":"'LD: status'"'""" ]
