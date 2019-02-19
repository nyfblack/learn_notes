# 3.OpenTSDB java api

OpenTSDB提供三种方式的读写操作：telnet、http、post，但官方并没提供JAVA版的API。但有开源贡献者**shifeng258** 编写了[opentsdb-client](https://github.com/shifeng258/opentsdb-client),方便大家使用JAVA操作OpenTSDB,下面是对OpenTSDB读写操作的一些封装.

## 1.OpentsdbClient(Opentsdb读写工具类)

~~~java
import org.opentsdb.client.ExpectResponse;
import org.opentsdb.client.HttpClient;
import org.opentsdb.client.HttpClientImpl;
import org.opentsdb.client.builder.MetricBuilder;
import org.opentsdb.client.request.Filter;
import org.opentsdb.client.request.Query;
import org.opentsdb.client.request.QueryBuilder;
import org.opentsdb.client.request.SubQueries;
import org.opentsdb.client.response.Response;
import org.opentsdb.client.response.SimpleHttpResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.*;

/**
 * Opentsdb读写工具类
 * <p/>
 */
public class OpentsdbClient {

    private static Logger log = LoggerFactory.getLogger(OpentsdbClient.class);

    /**
     * tagv的过滤规则: 精确匹配多项迭代值，多项迭代值以'|'分隔，大小写敏感
     */
    public static String FILTER_TYPE_LITERAL_OR = "literal_or";

    /**
     * tagv的过滤规则: 通配符匹配，大小写敏感
     */
    public static String FILTER_TYPE_WILDCARD = "wildcard";

    /**
     * tagv的过滤规则: 正则表达式匹配
     */
    public static String FILTER_TYPE_REGEXP = "regexp";

    /**
     * tagv的过滤规则: 精确匹配多项迭代值，多项迭代值以'|'分隔，忽略大小写
     */
    public static String FILTER_TYPE_ILITERAL_OR = "iliteral_or";

    /**
     * tagv的过滤规则: 通配符匹配，忽略大小写
     */
    public static String FILTER_TYPE_IWILDCARD = "iwildcard";

    /**
     * tagv的过滤规则: 通配符取非匹配，大小写敏感
     */
    public static String FILTER_TYPE_NOT_LITERAL_OR = "not_literal_or";

    /**
     * tagv的过滤规则: 通配符取非匹配，忽略大小写
     */
    public static String FILTER_TYPE_NOT_ILITERAL_OR = "not_iliteral_or";

    /**
     * tagv的过滤规则:
     * <p/>
     * Skips any time series with the given tag key, regardless of the value.
     * This can be useful for situations where a metric has inconsistent tag sets.
     * NOTE: The filter value must be null or an empty string
     */
    public static String FILTER_TYPE_NOT_KEY = "not_key";

    private HttpClient httpClient;

    public OpentsdbClient() {
        this.httpClient = new HttpClientImpl(ConfigLoader.getProperty("opentsdb.url"));
    }

    public OpentsdbClient(String opentsdbUrl) {
        this.httpClient = new HttpClientImpl(opentsdbUrl);
    }

    /**
     * 写入数据
     *
     * @param metric    指标
     * @param timestamp 时间点
     * @param value
     * @param tagMap
     * @return
     * @throws Exception
     */
    public boolean putData(String metric, Date timestamp, Long value, Map<String, String> tagMap) throws Exception {
        long timsSecs = timestamp.getTime();
        return this.putData(metric, timsSecs, value, tagMap);
    }

    /**
     * 写入数据
     *
     * @param metric    指标
     * @param timestamp 时间点
     * @param value
     * @param tagMap
     * @return
     * @throws Exception
     */
    public boolean putData(String metric, Date timestamp, Double value, Map<String, String> tagMap) throws Exception {
        long timsSecs = timestamp.getTime();
        return this.putData(metric, timsSecs, value, tagMap);
    }

    /**
     * 写入数据
     *
     * @param metric    指标
     * @param timestamp 转化为秒的时间点
     * @param value
     * @param tagMap
     * @return
     * @throws Exception
     */
    public boolean putData(String metric, long timestamp, Long value, Map<String, String> tagMap) throws Exception {
        MetricBuilder builder = MetricBuilder.getInstance();
        builder.addMetric(metric).setDataPoint(timestamp, value).addTags(tagMap);
        try {
            log.debug("write quest：{}", builder.build());
            Response response = httpClient.pushMetrics(builder, ExpectResponse.SUMMARY);
            log.debug("response.statusCode: {}", response.getStatusCode());
            return response.isSuccess();
        } catch (Exception e) {
            log.error("put data to opentsdb error: ", e);
            throw e;
        }
    }

    /**
     * 写入数据
     *
     * @param metric    指标
     * @param timestamp 转化为秒的时间点
     * @param value
     * @param tagMap
     * @return
     * @throws Exception
     */
    public boolean putData(String metric, long timestamp, Double value, Map<String, String> tagMap) throws Exception {
        MetricBuilder builder = MetricBuilder.getInstance();
        builder.addMetric(metric).setDataPoint(timestamp, value).addTags(tagMap);
        try {
            log.debug("write quest：{}", builder.build());
            Response response = httpClient.pushMetrics(builder, ExpectResponse.SUMMARY);
            log.debug("response.statusCode: {}", response.getStatusCode());
            return response.isSuccess();
        } catch (Exception e) {
            log.error("put data to opentsdb error: ", e);
            throw e;
        }
    }



    /**
     * 查询数据，返回的数据为json格式，结构为：
     * "[
     * "  {
     * "    metric: mysql.innodb.row_lock_time,
     * "    tags: {
     * "      host: web01,
     * "      dc: beijing
     * "    },
     * "    aggregateTags: [],
     * "    dps: {
     * "      1435716527: 1234,
     * "      1435716529: 2345
     * "    }
     * "  },
     * "  {
     * "    metric: mysql.innodb.row_lock_time,
     * "    tags: {
     * "      host: web02,
     * "      dc: beijing
     * "    },
     * "    aggregateTags: [],
     * "    dps: {
     * "      1435716627: 3456
     * "    }
     * "  }
     * "]";
     *
     * @param metric     要查询的指标
     * @param tagk       tagk
     * @param tagvFtype  tagv的过滤规则
     * @param tagvFilter tagv的匹配字符
     * @param aggregator 查询的聚合类型, 如: OpentsdbClient.AGGREGATOR_AVG, OpentsdbClient.AGGREGATOR_SUM
     * @param downsample 采样的时间粒度, 如: 1s,2m,1h,1d,2d
     * @param startTime  开始时间
     * @param endTime    结束时间
     * @return
     */
    public String getData(String metric, String tagk, String tagvFtype, String tagvFilter, String aggregator, String downsample,Date startTime, Date endTime) throws IOException {

        QueryBuilder queryBuilder = QueryBuilder.getInstance();
        Query query = queryBuilder.getQuery();

        query.setStart(startTime.getTime() / 1000);
        query.setEnd(endTime.getTime() / 1000);

        List<SubQueries> sqList = new ArrayList<SubQueries>();
        SubQueries sq = new SubQueries();
        sq.setMetric(metric);
        sq.setAggregator(aggregator);

        List<Filter> filters = new ArrayList<Filter>();
        Filter filter = new Filter();
        filter.setTagk(tagk);
        filter.setType(tagvFtype);
        filter.setFilter(tagvFilter);
        filter.setGroupBy(Boolean.TRUE);
        filters.add(filter);

        sq.setFilters(filters);

        sq.setDownsample(downsample + "-" + aggregator);

        sqList.add(sq);

        query.setQueries(sqList);

        try {
            log.debug("query request：{}", queryBuilder.build()); //这行起到校验作用
            SimpleHttpResponse spHttpResponse = httpClient.pushQueries(queryBuilder, ExpectResponse.DETAIL);
            log.debug("response.content: {}", spHttpResponse.getContent());

            if (spHttpResponse.isSuccess()) {
                return spHttpResponse.getContent();
            }
            return null;
        } catch (IOException e) {
            log.error("get data from opentsdb error: ", e);
            throw e;
        }
    }

    /**
     * 查询数据，返回的数据为json格式。
     *
     * @param metric     要查询的指标
     * @param filters     查询过滤的条件, 原来使用的tags在v2.2后已不适用
     *                   filter.setType(): 设置过滤类型, 如: wildcard, regexp
     *                   filter.setTagk(): 设置tag
     *                   filter.setFilter(): 根据type设置tagv的过滤表达式, 如: hqdApp|hqdWechat
     *                   filter.setGroupBy():设置成true, 不设置或设置成false会导致读超时
     * @param aggregator 查询的聚合类型, 如: OpentsdbClient.AGGREGATOR_AVG, OpentsdbClient.AGGREGATOR_SUM
     * @param downsample 采样的时间粒度, 如: 1s,2m,1h,1d,2d
     * @param startTime  开始时间
     * @param endTime    结束时间
     */
    public String getData(String metric, List<Filter> filters, String aggregator, String downsample,Date startTime, Date endTime) throws IOException {

        QueryBuilder queryBuilder = QueryBuilder.getInstance();
        Query query = queryBuilder.getQuery();

        query.setStart(startTime.getTime() / 1000);
        query.setEnd(endTime.getTime() / 1000);

        List<SubQueries> sqList = new ArrayList<SubQueries>();
        SubQueries sq = new SubQueries();
        sq.addMetric(metric);
        sq.addAggregator(aggregator);

        sq.setFilters(filters);

        sq.setDownsample(downsample + "-" + aggregator);
        sqList.add(sq);

        query.setQueries(sqList);

        try {
            log.info("query request：{}", queryBuilder.build()); //这行起到校验作用
            SimpleHttpResponse spHttpResponse = httpClient.pushQueries(queryBuilder, ExpectResponse.DETAIL);
            log.info("response.content: {}", spHttpResponse.getContent());

            if (spHttpResponse.isSuccess()) {
                return spHttpResponse.getContent();
            }
            return null;
        } catch (IOException e) {
            log.error("get data from opentsdb error: ", e);
            throw e;
        }
    }

    /**
     * 查询数据，返回tags与时序值的映射: Map<tags, Map<时间点, value>>
     *
     * @param metric     要查询的指标
     * @param tagk       tagk
     * @param tagvFtype  tagv的过滤规则
     * @param tagvFilter tagv的匹配字符
     * @param aggregator 查询的聚合类型, 如: OpentsdbClient.AGGREGATOR_AVG, OpentsdbClient.AGGREGATOR_SUM
     * @param downsample 采样的时间粒度, 如: 1s,2m,1h,1d,2d
     * @param startTime  开始时间
     * @param endTime    结束时间
     * @param retTimeFmt 返回的结果集中，时间点的格式, 如：yyyy-MM-dd HH:mm:ss 或 yyyyMMddHH 等
     * @return Map<tags, Map<时间点, value>>
     * @throws java.io.IOException
     */
    public Map<String, Map<String, Object>> getData(String metric, String tagk, String tagvFtype, String tagvFilter, String aggregator, String downsample,Date startTime, Date endTime, String retTimeFmt) throws IOException {
        String resContent = this.getData(metric, tagk, tagvFtype, tagvFilter, aggregator, 
                                         downsample, startTime, endTime);
        return this.convertContentToMap(resContent, retTimeFmt);
    }

    /**
     * 查询数据，返回tags与时序值的映射: Map<tags, Map<时间点, value>>
     *
     * @param metric     要查询的指标
     * @param filters     查询过滤的条件, 原来使用的tags在v2.2后已不适用
     *                   filter.setType(): 设置过滤类型, 如: wildcard, regexp
     *                   filter.setTagk(): 设置tag
     *                   filter.setFilter(): 根据type设置tagv的过滤表达式, 如: hqdApp|hqdWechat
     *                   filter.setGroupBy():设置成true, 不设置或设置成false会导致读超时
     * @param aggregator 查询的聚合类型, 如: OpentsdbClient.AGGREGATOR_AVG, OpentsdbClient.AGGREGATOR_SUM
     * @param downsample 采样的时间粒度, 如: 1s,2m,1h,1d,2d
     * @param startTime  开始时间
     * @param endTime    结束时间
     * @param retTimeFmt 返回的结果集中，时间点的格式, 如：yyyy-MM-dd HH:mm:ss 或 yyyyMMddHH 等
     * @return Map<tags, Map<时间点, value>>
     */
    public Map<String, Map<String, Object>> getData(String metric, List<Filter> filters, String aggregator, String downsample,Date startTime, Date endTime, String retTimeFmt) throws IOException {
        String resContent = this.getData(metric, filters, aggregator, downsample, startTime,
                                         endTime);
        return this.convertContentToMap(resContent, retTimeFmt);
    }


    public Map<String, Map<String, Object>> convertContentToMap(String resContent, String retTimeFmt) {

        // Map<tags, Map<时间点, value>>
        Map<String, Map<String, Object>> tagsValuesMap = 
            							new HashMap<String, Map<String, Object>>();

        if (resContent == null || "".equals(resContent.trim())) {
            return tagsValuesMap;
        }

        JSONArray array = (JSONArray) JSONObject.parse(resContent);
        if (array != null) {
            for (int i = 0; i < array.size(); i++) {
                JSONObject obj = (JSONObject) array.get(i);
                JSONObject tags = (JSONObject) obj.get("tags");
                JSONObject dps = (JSONObject) obj.get("dps");

                //timeValueMap.putAll(dps);
                Map<String, Object> timeValueMap = new HashMap<String, Object>();
                for (Iterator<String> it = dps.keySet().iterator(); it.hasNext(); ) {
                    String timstamp = it.next();
                    Date datetime = new Date(Long.parseLong(timstamp) * 1000);
                    timeValueMap.put(DateTimeUtil.format(datetime, retTimeFmt), 
                                     dps.get(timstamp));
                }
                tagsValuesMap.put(tags.toString(), timeValueMap);
            }
        }
        return tagsValuesMap;
    }
}
~~~

## 2.测试类

~~~java
import org.junit.Test;
import org.opentsdb.client.request.Filter;
import org.opentsdb.client.util.Aggregator;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;


public class OpentsdbClientTest {

    private static Logger log = LoggerFactory.getLogger(OpentsdbClientTest.class);

    @Test
    public void testPutData() {
        OpentsdbClient client = new OpentsdbClient(ConfigLoader.getProperty("opentsdb.url"));
        try {
            Map<String, String> tagMap = new HashMap<String, String>();
            tagMap.put("actv4", "union618");
            tagMap.put("ylff4", "hqdApp001");
            client.putData("pv", DateTimeUtil.parse("20160627 12:15", "yyyyMMdd HH:mm"), 
                           210l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 12:17", "yyyyMMdd HH:mm"), 
                           180l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 13:20", "yyyyMMdd HH:mm"), 
                           180l, tagMap);

            tagMap.clear();
            tagMap.put("actv4", "union618");
            tagMap.put("ylff4", "hqdWeixin001");
            client.putData("pv", DateTimeUtil.parse("20160627 12:15", "yyyyMMdd HH:mm"), 
                           100l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 12:17", "yyyyMMdd HH:mm"), 
                           150l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 13:20", "yyyyMMdd HH:mm"), 
                           150l, tagMap);

            tagMap.clear();
            tagMap.put("ylff4", "hqdWeixin001");
            client.putData("pv", DateTimeUtil.parse("20160627 12:15", "yyyyMMdd HH:mm"), 
                           40l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 12:17", "yyyyMMdd HH:mm"), 
                           50l, tagMap);
            client.putData("pv", DateTimeUtil.parse("20160627 13:20", "yyyyMMdd HH:mm"),
                           60l, tagMap);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Test
    public void testPutData2() {
        OpentsdbClient client = new OpentsdbClient(ConfigLoader.getProperty("opentsdb.url"));
        try {
            Map<String, String> tagMap = new HashMap<String, String>();
            tagMap.put("chl", "hqdWechat");

            client.putData("metric-t", DateTimeUtil.parse("20160627 12:25", "yyyyMMdd HH:mm"),
                           120l, tagMap);
            client.putData("metric-t", DateTimeUtil.parse("20160627 12:27", "yyyyMMdd HH:mm"),
                           810l, tagMap);
            client.putData("metric-t", DateTimeUtil.parse("20160627 13:20", "yyyyMMdd HH:mm"),
                           880l, tagMap);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Test
    public void testGetData() {
        OpentsdbClient client = new OpentsdbClient(ConfigLoader.getProperty("opentsdb.url"));
        try {
            Filter filter = new Filter();
            String tagk = "chl";
            String tagvFtype = OpentsdbClient.FILTER_TYPE_WILDCARD;
            String tagvFilter = "hqdapp*";

            Map<String, Map<String, Object>> tagsValuesMap = client.getData("metric-t", tagk, tagvFtype, tagvFilter, Aggregator.avg.name(), "1m",
                    DateTimeUtil.parse("2016-06-27 12:00:00", "yyyy-MM-dd HH:mm:ss"), DateTimeUtil.parse("2016-06-30 11:00:00", "yyyy-MM-dd HH:mm:ss"), "yyyyMMdd HHmm");

            for (Iterator<String> it = tagsValuesMap.keySet().iterator(); it.hasNext(); ) {
                String tags = it.next();
                System.out.println(">> tags: " + tags);
                Map<String, Object> tvMap = tagsValuesMap.get(tags);
                for (Iterator<String> it2 = tvMap.keySet().iterator(); it2.hasNext(); ) {
                    String time = it2.next();
                    System.out.println("    >> " + time + " <-> " + tvMap.get(time));
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Test
    public void testGetData2() {
        OpentsdbClient client = new OpentsdbClient(ConfigLoader.getProperty("opentsdb.url"));
        try {
            List<Filter> filters = new ArrayList<Filter>();
            Filter filter = new Filter();
            filter.setType(OpentsdbClient.FILTER_TYPE_LITERAL_OR);
            filter.setTagk("actv4");
            filter.setFilter("union618");
            filter.setGroupBy(Boolean.TRUE);
            filters.add(filter);

            filter = new Filter();
            filter.setType(OpentsdbClient.FILTER_TYPE_LITERAL_OR);
            filter.setTagk("ylff4");
            filter.setFilter("hqdApp001|hqdWeixin001");
            filter.setGroupBy(Boolean.TRUE);
            filters.add(filter);

            Map<String, Map<String, Object>> tagsValuesMap = client.getData("pv", filters, Aggregator.sum.name(), "1m",
                    DateTimeUtil.parse("2016-06-27 12:00:00", "yyyy-MM-dd HH:mm:ss"), DateTimeUtil.parse("2016-06-30 11:00:00", "yyyy-MM-dd HH:mm:ss"), "yyyyMMdd HHmm");

            for (Iterator<String> it = tagsValuesMap.keySet().iterator(); it.hasNext(); ) {
                String tags = it.next();
                System.out.println(">> tags: " + tags);
                Map<String, Object> tvMap = tagsValuesMap.get(tags);
                for (Iterator<String> it2 = tvMap.keySet().iterator(); it2.hasNext(); ) {
                    String time = it2.next();
                    System.out.println("    >> " + time + " <-> " + tvMap.get(time));
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
~~~
