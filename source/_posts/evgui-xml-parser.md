---
title: XML Parser for Libsvgtiny
date: 2020-04-08 21:41:00
updated: 2020-04-08 21:41:00
categories:
- 系统与底层
tags:
- svg
- svg tiny
- libsvgtiny
- embedded system
- vector
- gui
- twin
- window system
- cairo
- xml
- parser
- libxml2
- dom
- ezxml
---

这是我的[硕士论文笔记系列](https://blog.joouis.com/2020/evgui-design-review/)第三篇，原文写于 2016 年五月。

<!-- more -->



## Libxml2

### Libxml2 資料結構

```c
/**
 * xmlNode:
 *
 * A node in an XML tree.
 */

typedef struct _xmlNode xmlNode;
typedef xmlNode *xmlNodePtr;                                                                                                                      
struct _xmlNode {
    void           *_private;	/* application data */
    xmlElementType   type;	/* type number, must be second ! */
    const xmlChar   *name;      /* the name of the node, or the entity */
    struct _xmlNode *children;  /* parent->childs link */
    struct _xmlNode *last;	/* last child link */
    struct _xmlNode *parent;	/* child->parent link */
    struct _xmlNode *next;	/* next sibling link  */
    struct _xmlNode *prev;	/* previous sibling link  */
    struct _xmlDoc  *doc;	/* the containing document */

    /* End of common part */
    xmlNs           *ns;	/* pointer to the associated namespace */
    xmlChar         *content;	/* the content */
    struct _xmlAttr *properties;/* properties list */
    xmlNs           *nsDef;	/* namespace definitions on this node */
    void            *psvi;	/* for type/PSVI informations */
    unsigned short   line;	/* line number */
    unsigned short   extra;	/* extra data for XPath/XSLT */
};


/**
 * xmlDoc:
 *                                                                                                                                                
 * An XML document.
 */
typedef struct _xmlDoc xmlDoc;
typedef xmlDoc *xmlDocPtr;
struct _xmlDoc {
    void           *_private;	/* application data */
    xmlElementType  type;	/* XML_DOCUMENT_NODE, must be second ! */
    char           *name;	/* name/filename/URI of the document */
    struct _xmlNode *children;	/* the document tree */
    struct _xmlNode *last;	/* last child link */
    struct _xmlNode *parent;	/* child->parent link */
    struct _xmlNode *next;	/* next sibling link  */
    struct _xmlNode *prev;	/* previous sibling link  */
    struct _xmlDoc  *doc;	/* autoreference to itself */

    /* End of common part */
    int compression;		/* level of zlib compression */
    int standalone;		/* standalone document (no external refs)
                		¦ 1 if standalone="yes"
                		¦ 0 if standalone="no"
                		¦ -1 if there is no XML declaration
                		¦ -2 if there is an XML declaration, but no
                    			standalone attribute was specified */
    struct _xmlDtd  *intSubset; /* the document internal subset */
    struct _xmlDtd  *extSubset; /* the document external subset */
    struct _xmlNs   *oldNs;	/* Global namespace, the old way */
    const xmlChar  *version;	/* the XML version string */
    const xmlChar  *encoding;	/* external initial encoding, if any */
    void           *ids;	/* Hash table for ID attributes if any */
    void           *refs;	/* Hash table for IDREFs attributes if any */
    const xmlChar  *URL;	/* The URI for that document */
    int             charset;	/* encoding of the in-memory contentactually an
    				¦ xmlCharEncoding */
    struct _xmlDict *dict;	/* dict used to allocate names or NULL */
    void           *psvi;	/* for type/PSVI informations */
    int             parseFlags;	/* set of xmlParserOption used to parse the
    				¦ document */
    int             properties; /* set of xmlDocProperties for this document set
    				¦ at the end of parsing */
};


/**
 * xmlAttribute:
 *
 * An Attribute declaration in a DTD.
 */

typedef struct _xmlAttribute xmlAttribute;
typedef xmlAttribute *xmlAttributePtr;
struct _xmlAttribute {
    void		*_private;	/* application data */
    xmlElementType	type;	/* XML_ATTRIBUTE_DECL, must be second ! */
    const xmlChar	*name;	/* Attribute name */
    struct _xmlNode	*children;	/* NULL */
    struct _xmlNode	*last;	/* NULL */
    struct _xmlDtd	*parent;	/* -> DTD */
    struct _xmlNode	*next;	/* next sibling link  */
    struct _xmlNode	*prev;	/* previous sibling link  */
    struct _xmlDoc	*doc;	/* the containing document */

    struct _xmlAttribute	*nexth;	/* next in hash table */
    xmlAttributeType		atype;	/* The attribute type */
    xmlAttributeDefault		def;	/* the default */
    const xmlChar		*defaultValue;	/* or the default value */
    xmlEnumerationPtr		tree;	/* or the enumeration tree if any */
    const xmlChar		*prefix;	/* the namespace prefix if any */
    const xmlChar		*elem;	/* Element holding the attribute */
};
```

### Libxml2 構建 API

```c
dom_xml_parser *dom_xml_parser_create(void *dontCare1, void *dontCare2, MesgFuncPtr mesgFunc, void *dontCare3, dom_document **outDocument);
```

- 呼叫 `xmlInitParser()` 初始化 parser，然後使用 `calloc` 定義了 `dom_document` 和 `dom_xml_parser` 的指針各一個

```c
dom_xml_error dom_xml_parser_parse_chunk(dom_xml_parser *parser, const uint8_t *data, size_t len);
```

- 呼叫 `xmlReadMemory()` 函式解析

```c
dom_exception dom_document_get_document_element(dom_document *document, dom_element **outNode);
```

- 用來尋找根 `<svg>` ，實作直接呼叫 `xmlDocGetRootElement()` 函式得到 `document→node`，相當於默認是 SVG 檔案

```c
dom_exception dom_node_get_node_name(dom_node *node, dom_string **outString);
```

- 自幹一個 `dom_string_create_interned()` 函式
  - 如果 `svg→node→ns` 和 `svg→node→ns→prefix` 存在，呼叫 `asprintf(&qname, "%s:%s", n->ns->prefix, n->name)` 先把字串寫進 `qname`，然後再通過自幹的函式把 `qname` 給 `outString`
  - 若不存在就直接呼叫自幹函式把 `svg→node→name` 的值給 `outString`

```c
dom_exception dom_document_get_element_by_id(dom_node *node, dom_string *string, dom_element **outNode);
```

- 被 `svgtiny_gradient.c` 使用。先呼叫 `xmlHasProp()` 函式檢查 `document→node` 是否存在 `id` 這個屬性

  - 若 `id` 屬性存在就再檢查裏面是否存在需要的 ID 字串，就把 `gradient` 的指標指向這個 document
  - 內部還實作了一個 `getElementById()` 函式，供這個 API 調用，也是類似呼叫 `xmlHasProp()` 和做 `strcasecmp()`

```c
dom_exception dom_element_get_attribute(dom_node *node, dom_string *string, dom_string **outAttribute);
```

- 呼叫 `xmlHasProp()` 得到指定的屬性，例如讀 `svg` 的 `d` 屬性，也就是 path

```c
dom_exception dom_element_get_elements_by_tag_name(dom_element *element, dom_string *string, dom_nodelist **outNodeList);
```

- 被 `svgtiny_gradient.c` 使用。遍歷 `element→node→children` 鏈表，只要節點的名字和 tag 名相同，就把其加入 `nodeList` 裏

```c
dom_exception dom_node_get_node_type(dom_node *node, dom_node_type *outType);
```

- 需要注意 node type 的種類要滿足

```c
dom_xml_error dom_xml_parser_completed(dom_xml_parser *parser);
dom_xml_error dom_xml_parser_destroy(dom_xml_parser *parser);
dom_exception dom_nodelist_get_length(dom_nodelist *nodeList, uint32_t *outLen);
dom_exception dom_nodelist_item(dom_nodelist *nodeList, uint32_t index, dom_node **outItemp);
void dom_node_unref(dom_node *node);
void dom_nodelist_unref(dom_nodelist *nodeList);
dom_exception dom_node_get_first_child(dom_element *element, dom_element **outChild);
int dom_string_caseless_isequal(dom_string *a, dom_string *b);
dom_exception dom_node_get_next_sibling(dom_element *element, dom_element **outChild);
dom_exception dom_text_get_whole_text(dom_element *element, dom_string **outString);
lwc_error lwc_intern_string(const char *data, size_t len, lwc_string **outString);
int dom_string_caseless_lwc_isequal(dom_string *str, lwc_string *lwcString);
void lwc_string_unref(lwc_string *lwcString);
uint32_t dom_string_byte_length(dom_string *str);                        
dom_exception dom_string_create_interned(const uint8_t *data, size_t len, dom_string **outString);
char *dom_string_data(dom_string *str);
int dom_string_isequal(dom_string *a, dom_string *b);                    
dom_string *dom_string_ref(dom_string *str);
void dom_string_unref(dom_string *str);
```

- 實作簡單或未實作



## DOM Parser Choices

### [~~YXML~~](https://dev.yorhel.nl/yxml)

- No
  - 提供的接口函式太少
  - 使用 stack 替代 malloc

### [EZXML](https://github.com/lxfontes/ezxml)

- Try it!
  - 資料結構比較合理，包含基本常用之 node
  - 提供的 API 還算全面

- 成功解析 SVG 檔
  - 使用 `strcasecmp` 修復 `attr` 的大小寫問題，例如 “viewbox” “viewBox”
  - node 的 `type` 類型未初始化，除一開始 parse 到的 node 是 `DOM` 類型外，其餘 node 一律初始化爲 `ELEMENT` 類型

  ![](https://imxnua.bn.files.1drv.com/y4m5h5KUDF0dRdvanpRTVra33nMf4Em5Xoqp-Od2XAJf_Y1bfhjfk5fBTfvRmuwFV8HiOsOp1n56uO0M7CHF95oNeufgMXENidlY53MU07_P80y_AQoFKt9VDbkI0BQ5_qNXgKC_EEyaUS1Om-lJYcFQOqD4uaWhL7xLAwIhFrh64U4DagVVIGf2h2ch2-X1GeDSBdhom9tc55DfdQAmfu0BQ?width=1908&height=851&cropmode=none)

- 使用 libxml2 (左) 和 ezxml (右) 的對比圖

  ![](https://imxmua.bn.files.1drv.com/y4mx07uU18KkjdrT7Jrj1n7ELBos6O3OXrCY9HVNAVsOcAV8Z3ktgJ10Tg8OWRnVCSYbrL0YdaagSJj3dxjqsywhHXWHMyL7WpOKrrVghcs9sbT5xZ2M8RRUyE72XgI8WEbrWL5Q3YIC2jwZ2nkeyTYptf8hvbrhddMX-xxk9tNuKCWZj3n1Pr8P-kzIai-OvhopWtFvf1MNxL6U3Ky4hyVYw?width=1218&height=395&cropmode=none)

  ![](https://h8xeua.bn.files.1drv.com/y4mJAnzz49UoAqt6HmjwgZwToqgRWMq-SMfHrRAHntaDCzgOEXIIA2jtCCS2FQ-pGE87DSd_QmlTD_MCKzvbtUNaRo2nQpYwN-v0XqXYhVvpmdjieDqs0at6ily3G8Ek7w0YQwOmiBOM-h2AL-Hc0uEG8TTnHqahl_iUg1_bNkJ3KpqxqdA-5JfAugj1BAT0I_t2fi4v7YfnwECfQNU3c6bTQ?width=481&height=275&cropmode=none)

  ![](https://imxlua.bn.files.1drv.com/y4mDgFPvwwkVHE0SAGKfrU19u2brtadsA-Pzv5RoDBKvPGTVhG4cJ-1UyaDreqwblKvb35S6F9BhhLOt_m_cSBNi7qBlI6zUwVWCjEWiwAroDwrvYb5ii3y8XGOFLsdbBuGw8BbsOlIlBNjoI0HRQoX8gP_3UxugygYstN6Sy4bLIG2yDX5QBIaKN4OU1s0f3oFqKLFxTwnA5Rdsl5_-ouHTA?width=750&height=552&cropmode=none)

  ![](https://imxjua.bn.files.1drv.com/y4mH5176y3Kq-urCbkYRSQq3PZikoF3P6JO_zDKgCYudlBppGJgtzJMeP9-x1kQ1UP2Hmt16YSrJMPVYu2rKFgOjTxkqdVO29IZmNq3V6220rI1NFkvBX7oYAFKHtDj7OaQHrlNoMM7CujzRYpOUIMrTxPxpT4v5e1LoKTw6X5hSCMFoHPUj9MSNnaKksUqupUQxHl5gF8Z8Lzqba2o-qtqPw?width=1921&height=1038&cropmode=none)

  