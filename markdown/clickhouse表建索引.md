# clickhouse表建索引

alter table atom_v3_55_event add index key_index(`677`) type ngrambf_v1(3,512,2,0) granularity 4

optimize  table atom_v3_55_event  final

alter table atom_v3_43_event add index key_index(`564`) type ngrambf_v1(3,512,2,0) granularity 4

optimize  table atom_v3_43_event  final


alter table atom_v3_48_event add index key_index(`584`) type ngrambf_v1(3,512,2,0) granularity 4

optimize  table atom_v3_48_event  final

alter table atom_v3_57_event add index key_index(`749`) type ngrambf_v1(3,512,2,0) granularity 4

optimize  table atom_v3_57_event  final
