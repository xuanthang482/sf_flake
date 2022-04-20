## F1.2 SNOWFLAKE

### Connecting tới AWS S3

1. Tạo s3 bucket và upload json file lên S3 bucket vừa tạo
2. Tạo role s3FullAccess cho account để có quền đọc ghi trên s3 bucket đó. policy s3FullAccess sẽ có chi tiết trong file policy.json
3. Tạo storage integrations trong Snowflake với arn của account với role s3FullAccess, chỉ định các folder hoặc file mà s3 integration đó có thể truy cập. Sau khi tạo, chỉnh sửa policy bằng 2 attribure là storage_aws_role_arn và storage_aws_external_id để hoàn thành kết nối.

### Tạo Databse, Schema, Table

1. Tạo Database: CREATE OR REPLACE DATABASE Your_DB_name
2. Tạo các Schema:

- Schema EXTERNAL_STAGES chứa stage loading data từ s3
- Schema FILE_FORMATS chứa file format kiểu json.

### Loading và Transform data từ s3

1. Loading: Tạo SOURCE_TRANSFORMED_JSON table và dùng file format kiểu json để load data.
2. Transform:

- Transform cột **reviewTime** từ string sang Datetime.
- Thay đổi datatype cột **unixReviewTime** từ bigint datetime format yyyy-mm-dd.

### Sử dụng Task và Stream

1. Tạo các Procedure bao gồm :

- load_and_transformed_data_procedure : Tạo bảng source_table,loading data từ s3, transform và copy vào bảng source_table.
- merge_into_destination_table_procedure : merge data từ bảng source sang bảng destination.
- modify_source_table_data_procedure : insert thêm 1 record trên bảng source.

2. Tạo các Tasks bao gồm :

- load_and_transformed_data: gọi lại load_and_transformed_data_procedure.
- create_stream_on_source_table : tạo streams trên bảng source_table
- create_destination_table_by_cloning_source_table
- modify_source_table_data : gọi lại modify_source_table_data_procedure
- merge_into_destination_table: gọi lại merge_into_destination_table_procedure
- unload_transformed_data_into_s3 : upload data ngược lại s3

### SCD

SCD Type 1: ghi đè các record.

### Visualize data bằng Power BI

- Thống kê số lượng reviewer theo số lượng bài review (barchart)
- Thống kê số lượng bài reviewer theo năm (linechart)
- Thống kê ASIN theo năm (piechart)
