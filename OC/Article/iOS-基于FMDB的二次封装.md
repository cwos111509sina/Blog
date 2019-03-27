##iOS-基于FMDB的二次封装
<br>
一直对数据库恐之如鬼魅、惧之如虎狼。觉得是一种玄之又玄的技术，经过几天的摸索终于也算是对数据库稍有了解，基于FMDB进行了二次封装。

基于FMDB进行以下操作：

一、executeUpdate

1、建：create table if not exists t_xxxx (id integer primary key,name text,...)

2、增：insert into t_xxxx (name) values ('ssss');防止插入重复数据、我们对于重复数据进行替换操作(需设定主键，也就是上面的primary key)：insert or replace into t_xxxx (name) values ('ssss');

3、删：
      
      删除表中所有数据：delete from t_xxxx；

      删除表：drop table t_xxxx；
      
      特定条件删除：delete from t_xxxx where name in ('ssss','aaaa');删除name列值为ssss、aaaa的数据
      
4、改：update t_xxxx set name = 'ssss' where xxx = ‘xxx’；

二、executeQuery

查询所有数据：select * from t_xxx

根据行数查询：select * from t_xxx limit 2 offset 1;查询第二条数据开始后面的两条数据

inTransaction

事务处理：通过FMDBDatabaseQueue调用、提升效率（对比普通操作和database提升不是一星半点）


```
#import <Foundation/Foundation.h>

#import "FMDB.h"

NS_ASSUME_NONNULL_BEGIN


@interface BCFMDBObject : NSObject

+(instancetype)shareManager;

//插入数据、没有表时创建
-(void)fmdbInsertTable:(NSString *)table primaryKey:(NSString *)priKey vlaues:(NSArray *)values;

/**
 更新数据
 
 table:表名字 t_xxxx
 name :主键
 values : 包含改动字典的数组 字典内包含主键、修改内容键值对
 
 */
-(void)fmdbUpdateTable:(NSString *)table name:(NSString *)name values:(NSArray *)values;

/**
 删除表中数据
 
 table:表名字 t_xxxx
 name :主键
 values : 包含删除主键对应的值
 
 */
-(void)fmdbDeleteTable:(NSString *)table name:(NSString *)name values:(NSArray *)values;

/**
 获取表中数据
 table:表名字 t_xxxx
 limit:返回行数
 offset:从数据库取数据时条数的起点
 
 */

-(NSArray *)fmdbSelectFromTable:(NSString *)table;
-(NSArray *)fmdbSelectFromTable:(NSString *)table limit:(NSInteger)limit offset:(NSInteger)offset;

//删除表
-(void)fmdbDeleteTable:(NSString *)table;

//删除数据库
-(void)fmdbDelete;

```

.m文件内容
```

#import "BCFMDBObject.h"


@interface BCFMDBObject ()

@property (nonatomic,strong)FMDatabase *db;
@property (nonatomic,strong)FMDatabaseQueue *queue;

@end

@implementation BCFMDBObject


static BCFMDBObject *manager;
+(instancetype)shareManager{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (!manager) {
            manager = [[BCFMDBObject alloc]init];
            manager.queue = [FMDatabaseQueue databaseQueueWithPath:[NSString stringWithFormat:@"%@/park.sqlite",[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject]]];
            manager.db = [FMDatabase databaseWithPath:[NSString stringWithFormat:@"%@/park.sqlite",[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject]]];
            NSLog(@"filePath = %@",[NSString stringWithFormat:@"%@/park.sqlite",[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject]]);
        }
    });
    
    return manager;
}


+(instancetype)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (!manager) {
            manager = [super allocWithZone:zone];
        }
    });
    return manager;
}


//插入数据、没有表时创建
-(void)fmdbInsertTable:(NSString *)table primaryKey:(NSString *)priKey vlaues:(NSArray *)values{
    if (![self isCreateTable:table]) {
        NSDictionary * dict = values[0];
        
        NSMutableArray * arr = [dict allKeys].mutableCopy;
        [arr removeObject:priKey];
        NSString * keys = [arr componentsJoinedByString:@" text,"];
        [self.queue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
            
            [db executeUpdate:[NSString stringWithFormat:@"create table if not exists %@ (%@ text primary key,%@ text);",table,priKey,keys]];
        }];
    }
    [self.queue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
        
        for (NSDictionary * dict in values) {
            [db executeUpdate:[NSString stringWithFormat:@"insert or replace into %@ (%@) values (:%@)",table,[[dict allKeys] componentsJoinedByString:@","],[[dict allKeys] componentsJoinedByString:@",:"]] withParameterDictionary:dict];
        }
    }];
}
//更新数据
-(void)fmdbUpdateTable:(NSString *)table name:(NSString *)name values:(NSArray *)values{
    
    if (![self isCreateTable:table]) {
        return;
    }
    
    [self.queue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
        for (NSDictionary * dict in values) {
            NSMutableArray * arr = [[NSMutableArray alloc]init];
            for (NSString * key in [dict allKeys]) {
                if (![key isEqualToString:name]) {
                    [arr addObject:[NSString stringWithFormat:@"%@ = '%@'",key,dict[key]]];
                }
            }
            
            [db executeUpdate:[NSString stringWithFormat:@"update %@ set %@ where %@ = '%@'",table,[arr componentsJoinedByString:@" ' "],name,dict[name]]];
        }
    }];
    
    
}

//删除数据
-(void)fmdbDeleteTable:(NSString *)table{
    if ([_db open]) {
        [_db executeUpdate:[NSString stringWithFormat:@"delete from %@;",table]];
    }
}
-(void)fmdbDeleteTable:(NSString *)table name:(NSString *)name values:(NSArray *)values{
    
    if (![self isCreateTable:table]) {
        return;
    }
    
    [self.queue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
        
        NSString *sql = [NSString stringWithFormat:@"delete from %@ where %@ in ('%@')",table,name,[values componentsJoinedByString:[NSString stringWithFormat:@"','"]]];
        
        [db executeUpdate:sql];
    }];
    
    
}


//获取表中数据

-(NSArray *)fmdbSelectFromTable:(NSString *)table limit:(NSInteger)limit offset:(NSInteger)offset{
    NSMutableArray * array = [[NSMutableArray alloc]init];
    
    if ([_db open]) {
        
        FMResultSet *result = [_db executeQuery:[NSString stringWithFormat:@"select * from %@ limit %ld offset %ld;",table,limit,offset]];
        
        
        while ([result next]) {
            NSMutableDictionary * dict = [[NSMutableDictionary alloc]init];
            
            NSMutableArray * keyArray = [[NSMutableArray alloc]init];
            
            for (NSString *key in result.columnNameToIndexMap.allKeys) {
                [keyArray addObject:[result columnNameForIndex:[result.columnNameToIndexMap[key] intValue]]];
            }
            
            for (NSString *key in keyArray) {
                [dict setObject:[NSString stringWithFormat:@"%@",[result stringForColumn:key]] forKey:key];
            }
            [array addObject:dict];
        }
    }
    return array.copy;
}


-(NSArray *)fmdbSelectFromTable:(NSString *)table{
    NSMutableArray * array = [[NSMutableArray alloc]init];
    
    if ([_db open]) {
        
        FMResultSet *result = [_db executeQuery:[NSString stringWithFormat:@"select * from %@",table]];
        
        
        while ([result next]) {
            NSMutableDictionary * dict = [[NSMutableDictionary alloc]init];
            
            NSMutableArray * keyArray = [[NSMutableArray alloc]init];
            
            for (NSString *key in result.columnNameToIndexMap.allKeys) {
                [keyArray addObject:[result columnNameForIndex:[result.columnNameToIndexMap[key] intValue]]];
            }
            
            for (NSString *key in keyArray) {
                [dict setObject:[NSString stringWithFormat:@"%@",[result stringForColumn:key]] forKey:key];
            }
            [array addObject:dict];
        }
    }
    return array.copy;
}

-(BOOL)isCreateTable:(NSString *)table{
    __block BOOL isCreat = NO;
    
    [self.queue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
        FMResultSet *result = [db executeQuery:@"select count(*) as 'count' from sqlite_master where type = 'table' and name = ?",table];
        if ([result next]) {
            NSInteger count = [result intForColumn:@"count"];
            if (count > 0) {
                isCreat = YES;
            }
        }
    }];
    return NO;
}

-(void)fmdbDelete{
    
    NSString *filePath = [NSString stringWithFormat:@"%@/park.sqlite",[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject]];
    
    NSFileManager *manager = [NSFileManager defaultManager];
    
    if ([manager fileExistsAtPath:filePath]) {
        [manager removeItemAtPath:filePath error:nil];
    }
}



@end

```


[项目链接](https://github.com/cwos111509sina/BCFMDBTest.git)

[返回首页](https://cwos111509sina.github.io/Blog/)

