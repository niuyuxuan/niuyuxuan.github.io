---
title: python插件
date: 2024-10-11
categories:
  - 开发笔记
tags: 
- python
- Elasticsearch
- 代码片段
---


1.Elasticsearch查询导出数据，常用代码

```python
# coding=utf-8
import elasticsearch
import warnings

warnings.filterwarnings('ignore')


class Es:
    def __init__(self):
        # es链接
        self.es = elasticsearch.Elasticsearch(hosts='', verify_certs=False)
        self.scroll_id = None

    def first_query(self):
        result = self.es.search(
            # 相应的索引
            index='',
            query=query,
            size=1000,
            source_includes=['message'],
            scroll='1m',
            request_timeout=200
        )
        self.scroll_id = result['_scroll_id']
        return result['hits']['hits']

    def scroll_query(self):
        result = self.es.scroll(
            scroll_id=self.scroll_id,
            scroll='1m',
            request_timeout=200
        )
        self.scroll_id = result['_scroll_id']
        return result['hits']['hits']


def get_data() -> iter:
    """
    查询
    处理原始日志，message的字段
    :return:
    """
    data_list = []
    # 提取es的message数据
    doc_list = es.first_query()
    # 滚动查询
    while doc_list:
        # 请根据需要的字段来提取数据
        data_list.extend([x['_source']['message'] for x in doc_list])
        doc_list = es.scroll_query()
        print('日志数量', len(data_list))


if __name__ == '__main__':
    es = Es()
    # 查询es
    query = {

    }
```

2.创建，删除数据索引

```python
# coding=utf-8
import elasticsearch
import warnings

warnings.filterwarnings('ignore')


def create_index():
    es.indices.create(
        index=index,
        settings={
            'number_of_shards': 1,
            'number_of_replicas': 1
        },
        request_timeout=100
    )

def delete_index():
    es.indices.delete(
        index=index,
        request_timeout=100
    )

if __name__ == '__main__':
    es = elasticsearch.Elasticsearch(hosts='',
                                     verify_certs=False)

    # 数据索引
    index = ''
    create_index()
    # delete_index()
```

3.将多个列表整合为一个列表，获取列表中所有字符串

```python
def get_string_list(lst):
    """
    获取列表中的所有字符串
    """
    result = []
    for item in lst:
        if isinstance(item, (list, tuple)):
            result.extend(get_string_list(item))
        else:
            result.append(item)
    return result
```

4.Elasticsearch上传数据，常用代码

```python
import elasticsearch.helpers
import warnings

warnings.filterwarnings('ignore')


class ES:
    def __init__(self, es_host, verify_certs=False):
        self.es = elasticsearch.Elasticsearch(hosts=es_host, verify_certs=verify_certs)

    def upload_doc_list(self, doc_list):
        result = elasticsearch.helpers.bulk(self.es, doc_list, request_timeout=200)
        return result


def upload_rule_info():
    data_list = []

    # info添加主要数据，_source,_index等
    info = {
        '_index': '',
    }
    data_list.append(info)

    es.upload_doc_list(data_list)


if __name__ == '__main__':
    # es链接
    HOSTS_180 = ['']
    es = ES(HOSTS_180)
    # 可以根据需求改写，比如批量上传
    upload_rule_info()
```

5.将每个IP地址与子网掩码24进行组合，生成一个新的IP网段
