#### 1.�Ķ��ĵ���������Ϣ
- �ƶ�Ӧ��΢��֧���̻�����ָ���ĵ�(����΢��������д��Ϣ,�����̻�ID):
https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317780&token=&lang=zh_CN
- �����ĵ�:
https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1
- �������߰���SDK���أ�
 https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167&lang=zh_CN

>��������Ҫ��������Ϣ�õ��̻�Id�����̻�ƽ̨���ɵ���Կ��AppId��AppSecret��

һ�㽫��Щ��Ϣд��һ�����������淽��ά�������£�
```java
   /**
    * ΢��֧���ر�����
    */
    public calss WeChatConstants{

         public static final String WECHAT_APP_ID = "��΢�ſ���ƽ̨���ɵ�AppId";

         public static final String WECHAT_APP_SECRET = "��΢�ſ���ƽ̨���ɵ�AppSecret";

         public static final String WECHAT_PARTNER_ID = "�̻�ID";

         public static final String WECHAT_PARTNER_KEY = "���̻�ƽ̨���ɵ���Կ";

         public static final String WECHAT_RESULT_NOTIFY = "΢�Żص��������ӿڵ�ַ";

         public static final String WECHAT_UNIFIED_ORDER = "΢��ͳһ�µ��ӿڵ�ַ";
    }
```
�ɸ�����Ŀ�����滻���ϲ���ֵ���Լ�ʹ�á�

- �Զ���΢��֧��model ������������á�

```javascript
   /**
    * ���� ��ֻ��Ҫ���� key, value ���ɡ�
    */
    public class WeChatPayBean<K, V> {

        private K key;

        private V value;

        public WeChatPayBean(K key, V value) {
            this.key = key;
            this.value = value;
        }
    
        public K getKey() {
            return key;
        }

        public void setKey(K key) {
            this.key = key;
        }

        public V getValue() {
            return value;
        }

        public void setValue(V value) {
            this.value = value;
        }
    }
```
#### 2.��ʼ֧��
> �����̻�ϵͳ�ȵ��øýӿ���΢��֧�������̨����Ԥ֧�����׵���������ȷ��Ԥ֧�����׻ػ���ʶ������APP�������֧����Ҳ����˵����˵��øýӿڴ�΢���Ǳ߻��һ����֧�������š�

- ʹ��AsyncTask��ȡ������Ϣ
- ΢��֧��ͳһ�µ��ӿڵ�ַ:https://api.mch.weixin.qq.com/pay/unifiedorder

```java
    private Map<String, String> mResult ;// ����һ��Map���ϣ��������յõ��Ķ�����Ϣ
    
    public void getPrepayId(final Activity activity, final IWXAPI mApi) {
        Toast.makeTextactivity, "��ȡ������...", Toast.LENGTH_SHORT).show();
        new AsyncTask<Void, Void, Map<String, String>>() {

            @Override
            protected void onPostExecute(Map<String, String> result) {
                if (result != null) {
                    mResult = result;
                    getPayReq(mApi);
                }
            }

            @Override
            protected Map<String, String> doInBackground(Void... voids) {
                String url = String.format(WeChatConstants.WECHAT_UNIFIED_ORDER);// ͳһ�µ��ӿڵ�ַ
                byte[] buf = HttpUtils.httpPost(url, getProductArgs());// HttpUtils.httpPost(); Ϊpost����
                try {
                    return decodeXml(new String(buf));
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;
            }
        }.execute();
    }
```
- �������֧��

```java
    private void getPayReq(IWXAPI mApi) {
        PayReq mPayReq = new PayReq();
        mPayReq.appId = WeChatConstants.WECHAT_APP_ID;// AppId
        mPayReq.partnerId = WeChatConstants.WECHAT_PARTNER_ID;// ΢��֧��������̻���
        mPayReq.prepayId = mResult.get("prepay_id");// ֧������Id
        mPayReq.packageValue = "Sign=WXPay";// ��չ�ֶΣ�����д�̶�ֵSign=WXPay
        mPayReq.nonceStr = getNonceString();// ������ַ���
        mPayReq.timeStamp = String.valueOf(getTimeStamp());// ʱ���
        List<WeChatPayBean> signParams = new AraaryList<WeChatPayBean>();
        signParams.add(new WeChatPayBean("appid", mPayReq.appId));
        signParams.add(new WeChatPayBean("noncestr", mPayReq.nonceStr));
        signParams.add(new WeChatPayBean("package", mPayReq.packageValue));
        signParams.add(new WeChatPayBean("partnerid", mPayReq.partnerId));
        signParams.add(new WeChatPayBean("prepayid", mPayReq.prepayId));
        signParams.add(new WeChatPayBean("timestamp", mPayReq.timeStamp));
        mPayReq.sign = genAppSign(signParams);
        mApi.sendReq(mPayReq);
    }
```
- ͳһ�µ������������(����ֻ���ñ�Ҫ����)��

```java
   /**
    * ����΢�Źٷ�֧���ĵ�������Ҫ�Ĳ���
    */
    private String getProductArgs() {  
        StringBuffer xml = new StringBuffer();
        try {
            String nonceString = getNonceString();// ��ȡ����ַ����������ڱ����������
            String bodyString = "xxx��ֵ";// ��Ʒ�����������������磺��Ѷ��ֵ����-QQ��Ա��ֵ
            xml.append("</xml>");
            List<WeChatPayBean> packageParams = new ArrayList<WeChatPayBean>();
            packageParams.add(new WeChatPayBean("appid", WeChatConstants.WECHAT_APP_ID));// AppId
            packageParams.add(new WeChatPayBean("body", bodyString));
            packageParams.add(new WeChatPayBean("mch_id", WeChatConstants.WECHAT_PARTNER_ID));// ΢��֧��������̻���
            packageParams.add(new WeChatPayBean("nonce_str", nonceString));// ����ַ���
            packageParams.add(new WeChatPayBean("notify_url", WeChatConstants.WECHAT_RESULT_NOTIFY ));// ΢�Żص�����˵�ַ
            packageParams.add(new WeChatPayBean("out_trade_no", "֧��������"));// ����˷��ص�֧��������
            packageParams.add(new WeChatPayBean("spbill_create_ip", AppUtils.getPhoneIP(AppContext.instance)));
            packageParams.add(new WeChatPayBean("total_fee", "�������"));// �����ܽ���λΪ�֡������̻���Ʒ�۸��壬�˴�����������
            packageParams.add(new WeChatPayBean("trade_type", "APP"));// ��������
            packageParams.add(new WeChatPayBean("sign", getAppSign(packageParams)));// ǩ���������ڱ�������
            String xmls = toXml(packageParams);
            return new String(xmls.toString().getBytes(), "ISO8859-1");// ���ñ����ʽ
        } catch (Exception e) {
            return null;
        }
    }
```
- �õ�һ������ַ���

```java
    private String getNonceString() {
        Random random = new Random();
        return MD5.getMessageDigest(String.valueOf(random.nextInt(10000)).getBytes());
    }
```
- ΢��֧��MD5���ܹ�����

```java
   public class MD5 {    
            
       private MD5() { }    
            
       public final static String getMessageDigest(byte[] buffer) {
            char hexDigits[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};        
            try {            
                    MessageDigest mdTemp = MessageDigest.getInstance("MD5");
                    mdTemp.update(buffer);
                    byte[] md = mdTemp.digest();
                    int j = md.length;
                    char str[] = new char[j * 2];
                    int k = 0;
                    for (int i = 0; i < j; i++) {
                          byte byte0 = md[i];
                          str[k++] = hexDigits[byte0 >>> 4 & 0xf];
                          str[k++] = hexDigits[byte0 & 0xf];
                    }
                    return new String(str);
            } catch (Exception e) {
                    return null;
            }
        }
   }
```
- ǩ��
>ǩ���淶�ĵ�:https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3

```java
    private String getAppSign(List<PayBean> params) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < params.size(); i++) {
            sb.append(params.get(i).getKey());
            sb.append('=');
            sb.append(params.get(i).getValue());
            sb.append('&');
        }
        sb.append("key=");
        sb.append(WeChatConstants.WECHAT_PARTNER_KEY);// �̻�ƽ̨���ɵ���Կ
        String appSign = MD5.getMessageDigest(sb.toString().getBytes()).toUpperCase();
        return appSign;
    }
```
- ʱ���

```java
    private long getTimeStamp() {
        SimpleDateFormat time = new SimpleDateFormat("yyyyMMddkkmmss");
        return Long.valueOf(time.format(new Date()));
    }
```
- ����XML

```java
    private String toXml(List<WeChatPayBean> params) {
        StringBuilder sb = new StringBuilder();
        sb.append("<xml>");
        for (int i = 0; i < params.size(); i++) {
            sb.append("<" + params.get(i).getKey() + ">");
            sb.append(params.get(i).getValue());
            sb.append("</" + params.get(i).getKey() + ">");
        }
        sb.append("</xml>");
        return sb.toString();
    }
```
- ����XML

```java
    public Map<String, String> decodeXml(String content) {
        try {
            Map<String, String> xml = new HashMap<String, String>();
            XmlPullParser parser = Xml.newPullParser();
            parser.setInput(new StringReader(content));
            int event = parser.getEventType();
            while (event != XmlPullParser.END_DOCUMENT) {
                String nodeName = parser.getName();
                switch (event) {
                    case XmlPullParser.START_DOCUMENT:
                        break;
                    case XmlPullParser.START_TAG:
                        if ("xml".equals(nodeName) == false) {
                            xml.put(nodeName, parser.nextText());
                        }
                        break;
                    case XmlPullParser.END_TAG:
                        break;
                }
                event = parser.next();
            }
            return xml;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```
#### 3.΢��֧���ۿ�ɹ���΢�Ż�ͨ���ص������̻���̨�ۿ�ɹ�����ʱ������Ҫ���ú�̨�ӿڲ�ѯ����״̬��