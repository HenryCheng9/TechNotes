item_sum.h#439

```cpp
enum Sumfunctype {  
  COUNT_FUNC,           // COUNT  
  COUNT_DISTINCT_FUNC,  // COUNT (DISTINCT)  
  SUM_FUNC,             // SUM  
  SUM_DISTINCT_FUNC,    // SUM (DISTINCT)  
  AVG_FUNC,             // AVG  
  AVG_DISTINCT_FUNC,    // AVG (DISTINCT)  
  MIN_FUNC,             // MIN  
  MAX_FUNC,             // MAX  
  STD_FUNC,             // STD/STDDEV/STDDEV_POP  
  VARIANCE_FUNC,        // VARIANCE/VAR_POP and VAR_SAMP  
  SUM_BIT_FUNC,         // BIT_AND, BIT_OR and BIT_XOR  
  UDF_SUM_FUNC,         // user defined functions  
  GROUP_CONCAT_FUNC,    // GROUP_CONCAT  
  JSON_AGG_FUNC,        // JSON_ARRAYAGG and JSON_OBJECTAGG  
  ROW_NUMBER_FUNC,      // Window functions  
  RANK_FUNC,  
  DENSE_RANK_FUNC,  
  CUME_DIST_FUNC,  
  PERCENT_RANK_FUNC,  
  NTILE_FUNC,  
  LEAD_LAG_FUNC,  
  FIRST_LAST_VALUE_FUNC,  
  NTH_VALUE_FUNC,  
  ROLLUP_SUM_SWITCHER_FUNC,  
  GEOMETRY_AGGREGATE_FUNC  
};
```

dataset



mysqld.trace

```cpp
/**  
  The pass-through aggregator.  Implements AGGFN (DISTINCT ..) by knowing it gets distinct data on input.  So it just pumps them back to the Item_sum descendant class.*/  
class Aggregator_simple : public Aggregator {  
 public:  
  Aggregator_simple(Item_sum *sum) : Aggregator(sum) {}  
  Aggregator_type Aggrtype() override { return    Aggregator::SIMPLE_AGGREGATOR; }  
  
  bool setup(THD *thd) override { return item_sum->setup(thd); }  
  void clear() override { item_sum->clear(); }  
  bool add() override { return item_sum->add(); }  
  void endup() override {}  
  my_decimal *arg_val_decimal(my_decimal *value) override;  
  double arg_val_real() override;  
  bool arg_is_null(bool use_null_value) override;  
};
```

```cpp
class Aggregator_distinct : public Aggregator {  
  friend class Item_sum_sum;  
  bool endup_done;  
  TABLE *table;  
  uint32 *field_lengths;    
  Temp_table_param *tmp_table_param;  
  Unique *tree;  
  uint tree_key_length;  
  
  enum Const_distinct {  
    NOT_CONST = 0,  
    CONST_NULL,  
    CONST_NOT_NULL  
  } const_distinct;  
  
  bool use_distinct_values;  
  
 public:  
  Aggregator_distinct(Item_sum *sum)  
      : Aggregator(sum),  
        table(nullptr),  
        tmp_table_param(nullptr),  
        tree(nullptr),  
        const_distinct(NOT_CONST),  
        use_distinct_values(false) {}  
  ~Aggregator_distinct() override;  
  Aggregator_type Aggrtype() override { return DISTINCT_AGGREGATOR; }  
  
  bool setup(THD *) override;  
  void clear() override;  
  bool add() override;  
  void endup() override;  
  my_decimal *arg_val_decimal(my_decimal *value) override;  
  double arg_val_real() override;  
  bool arg_is_null(bool use_null_value) override;  
  
  bool unique_walk_function(void *element);  
  static int composite_key_cmp(const void *arg, const void *a, const void *b);  
};
```

从上看 Aggregator_simple 基本只是个调⽤用wrapper，表示⾮非distinct的Item_sum处理理，因此直接调⽤用Item_sum的逻辑即可。 

在MySQL的实现中⼀一个聚合函数Item_sum的步骤简单就是三步:setup, add, endup。 

setup在处理理之前初始化，add表示每条record的process，endup就是收尾最后计算聚合的结果。

## setup

```
T@9: | | | | | | | | >JOIN::make_tmp_tables_info
T@9: | | | | | | | | | >make_group_fields
T@9: | | | | | | | | | <make_group_fields
T@9: | | | | | | | | | >JOIN::make_sum_func_list
T@9: | | | | | | | | | <JOIN::make_sum_func_list
T@9: | | | | | | | | | >setup_sum_funcs
T@9: | | | | | | | | | | >count_field_types
T@9: | | | | | | | | | | <count_field_types
T@9: | | | | | | | | | | >*create_tmp_table
T@9: | | | | | | | | | | | >*MEM_ROOT::AllocSlow
T@9: | | | | | | | | | | | | enter: root: 0x179cbfd70
T@9: | | | | | | | | | | | | >*MEM_ROOT::AllocBlock
T@9: | | | | | | | | | | | | <*MEM_ROOT::AllocBlock
T@9: | | | | | | | | | | | <*MEM_ROOT::AllocSlow
T@9: | | | | | | | | | | | >*fn_format
T@9: | | | | | | | | | | | | enter: name: #sqlb426_9_3  dir: /var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/  extension:   flag: 6
T@9: | | | | | | | | | | | | >dirname_part
T@9: | | | | | | | | | | | | | enter: '#sqlb426_9_3'
T@9: | | | | | | | | | | | | | >*convert_dirname
T@9: | | | | | | | | | | | | | <*convert_dirname
T@9: | | | | | | | | | | | | <dirname_part
T@9: | | | | | | | | | | | | >*convert_dirname
T@9: | | | | | | | | | | | | <*convert_dirname
T@9: | | | | | | | | | | | | >unpack_dirname
T@9: | | | | | | | | | | | | | >normalize_dirname
T@9: | | | | | | | | | | | | | | >dirname_part
T@9: | | | | | | | | | | | | | | | enter: '/var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/'
T@9: | | | | | | | | | | | | | | | >*convert_dirname
T@9: | | | | | | | | | | | | | | | <*convert_dirname
T@9: | | | | | | | | | | | | | | <dirname_part
T@9: | | | | | | | | | | | | | | >cleanup_dirname
T@9: | | | | | | | | | | | | | | | enter: from: '/var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/'
T@9: | | | | | | | | | | | | | | | exit: to: '/var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/'
T@9: | | | | | | | | | | | | | | <cleanup_dirname
T@9: | | | | | | | | | | | | | <normalize_dirname
T@9: | | | | | | | | | | | | <unpack_dirname
T@9: | | | | | | | | | | | | >strlength
T@9: | | | | | | | | | | | | <strlength
T@9: | | | | | | | | | | | <*fn_format
T@9: | | | | | | | | | | | >init_tmp_table_share
T@9: | | | | | | | | | | | | enter: table: ''.'/var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/#sqlb426_9_3'
T@9: | | | | | | | | | | | | >MEM_ROOT::Clear
T@9: | | | | | | | | | | | | | enter: root: 0x13401d8f8
T@9: | | | | | | | | | | | | <MEM_ROOT::Clear
T@9: | | | | | | | | | | | <init_tmp_table_share
T@9: | | | | | | | | | | | >*create_tmp_field
T@9: | | | | | | | | | | | <*create_tmp_field
T@9: | | | | | | | | | | | info: hidden_field_count: 0
T@9: | | | | | | | | | | | >plugin_lock
T@9: | | | | | | | | | | | | >intern_plugin_lock
T@9: | | | | | | | | | | | | | info: thd: 0x124de4e00, plugin: "TempTable", ref_count: 2
T@9: | | | | | | | | | | | | <intern_plugin_lock
T@9: | | | | | | | | | | | <plugin_lock
T@9: | | | | | | | | | | | >*get_new_handler
T@9: | | | | | | | | | | | | enter: alloc: 0x124de7680
T@9: | | | | | | | | | | | | info: handler created F_UNLCK 2 F_RDLCK 1 F_WRLCK 3
T@9: | | | | | | | | | | | | >::handler::Table_flags temptable::Handler::table_flags
T@9: | | | | | | | | | | | | | temptable_api: this=0x13401abd8; return=4303365297
T@9: | | | | | | | | | | | | <::handler::Table_flags temptable::Handler::table_flags
T@9: | | | | | | | | | | | <*get_new_handler
T@9: | | | | | | | | | | | >ha_key_alg temptable::Handler::get_default_index_algorithm
T@9: | | | | | | | | | | | <ha_key_alg temptable::Handler::get_default_index_algorithm
T@9: | | | | | | | | | | | >bitmap_init
T@9: | | | | | | | | | | | <bitmap_init
T@9: | | | | | | | | | | | >bitmap_init
T@9: | | | | | | | | | | | <bitmap_init
T@9: | | | | | | | | | | | >bitmap_init
T@9: | | | | | | | | | | | <bitmap_init
T@9: | | | | | | | | | | | >create_tmp_table_with_fallback
T@9: | | | | | | | | | | | | >int temptable::Handler::create
T@9: | | | | | | | | | | | | | temptable_api_kv_store: this=0x1053b3180 size=0; bucket_count=0 load_factor=0.000000 max_load_factor=1.000000 max_bucket_count=54901024028897475
T@9: | | | | | | | | | | | | | >char *PFS_instr_name::str
T@9: | | | | | | | | | | | | | <char *PFS_instr_name::str
T@9: | | | | | | | | | | | | | temptable_allocator: block create: size=1048576, new_block=(address=0x2f8018020, size=1048576, num_chunks=0, first_pristine=32)
T@9: | | | | | | | | | | | | | temptable_allocator: allocate from block: chunk_size=16, from_block=(address=0x2f8018020, size=1048576, num_chunks=1, first_pristine=56); return=0x2f8018048
T@9: | | | | | | | | | | | | | temptable_allocator: allocate from block: chunk_size=16, from_block=(address=0x2f8018020, size=1048576, num_chunks=2, first_pristine=80); return=0x2f8018060
T@9: | | | | | | | | | | | | | temptable_allocator: allocate from block: chunk_size=664, from_block=(address=0x2f8018020, size=1048576, num_chunks=3, first_pristine=752); return=0x2f8018078
T@9: | | | | | | | | | | | | | temptable_allocator: allocate from block: chunk_size=8192, from_block=(address=0x2f8018020, size=1048576, num_chunks=4, first_pristine=8952); return=0x2f8018318
T@9: | | | | | | | | | | | | | temptable_allocator: allocate from block: chunk_size=24, from_block=(address=0x2f8018020, size=1048576, num_chunks=5, first_pristine=8984); return=0x2f801a320
T@9: | | | | | | | | | | | | <int temptable::Handler::create
T@9: | | | | | | | | | | | <create_tmp_table_with_fallback
T@9: | | | | | | | | | | | >handler::ha_open
T@9: | | | | | | | | | | | | enter: name: /var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/#sqlb426_9_3  db_type: 29  db_stat: 3  mode: 2  lock_test: 516
T@9: | | | | | | | | | | | | info: old m_lock_type: 2 F_UNLCK 2
T@9: | | | | | | | | | | | | >int temptable::Handler::open
T@9: | | | | | | | | | | | | | temptable_api: this=0x13401abd8 /var/folders/zk/y2w71h5j1xv0rmcl1stvnbs80000gn/T/#sqlb426_9_3; return=OK
T@9: | | | | | | | | | | | | <int temptable::Handler::open
T@9: | | | | | | | | | | | | >::handler::Table_flags temptable::Handler::table_flags
T@9: | | | | | | | | | | | | | temptable_api: this=0x13401abd8; return=4303365297
T@9: | | | | | | | | | | | | <::handler::Table_flags temptable::Handler::table_flags
T@9: | | | | | | | | | | | <handler::ha_open
T@9: | | | | | | | | | | | opt: (null): starting struct
T@9: | | | | | | | | | | | opt: creating_tmp_table: starting struct
T@9: | | | | | | | | | | | opt: tmp_table_info: starting struct
T@9: | | | | | | | | | | | opt: table: "intermediate_tmp_table"
T@9: | | | | | | | | | | | opt: columns: 1
T@9: | | | | | | | | | | | opt: row_length: 32
T@9: | | | | | | | | | | | opt: key_length: 32
T@9: | | | | | | | | | | | opt: unique_constraint: 0
T@9: | | | | | | | | | | | opt: makes_grouped_rows: 0
T@9: | | | | | | | | | | | opt: cannot_insert_duplicates: 1
T@9: | | | | | | | | | | | opt: location: "TempTable"
T@9: | | | | | | | | | | | opt: tmp_table_info: ending struct
T@9: | | | | | | | | | | | opt: creating_tmp_table: ending struct
T@9: | | | | | | | | | | | opt: (null): ending struct
T@9: | | | | | | | | | | | >MEM_ROOT::Clear
T@9: | | | | | | | | | | | | enter: root: 0x179cbfd70
T@9: | | | | | | | | | | | <MEM_ROOT::Clear
T@9: | | | | | | | | | | <*create_tmp_table
T@9: | | | | | | | | | | >init_tree
T@9: | | | | | | | | | | | enter: tree: 0x13401bdd0  element_size: 31
T@9: | | | | | | | | | | <init_tree
T@9: | | | | | | | | | | >open_cached_file
T@9: | | | | | | | | | | | >init_io_cache_ext
T@9: | | | | | | | | | | | | enter: cache: 0x13401bc88  type: 2  pos: 0
T@9: | | | | | | | | | | | | info: init_io_cache: cachesize = 65536
T@9: | | | | | | | | | | | <init_io_cache_ext
T@9: | | | | | | | | | | <open_cached_file
T@9: | | | | | | | | | <setup_sum_funcs
T@9: | | | | | | | | | >JOIN::get_end_select_func
T@9: | | | | | | | | | | info: Using end_send_group
T@9: | | | | | | | | | <JOIN::get_end_select_func
T@9: | | | | | | | | <JOIN::make_tmp_tables_info
```