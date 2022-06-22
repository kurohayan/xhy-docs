Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.7.22</version>
    </dependency>


示例程序
------------------

java::


    import cn.hutool.core.io.IoUtil;
    import cn.hutool.core.util.IdUtil;
    import cn.hutool.http.HttpRequest;
    import cn.hutool.http.HttpResponse;
    import cn.hutool.http.HttpUtil;
    import cn.hutool.json.JSON;
    import cn.hutool.json.JSONUtil;
    import org.junit.Test;

    import java.io.*;
    import java.nio.charset.StandardCharsets;
    import java.security.KeyFactory;
    import java.security.NoSuchAlgorithmException;
    import java.security.PrivateKey;
    import java.security.Signature;
    import java.security.spec.InvalidKeySpecException;
    import java.security.spec.PKCS8EncodedKeySpec;
    import java.util.*;
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;

    public class ApiRequestTest {

        static class PemReader extends BufferedReader {
            private static final String BEGIN = "-----BEGIN ";
            private static final String END = "-----END ";
            private static final String END_BOUNDARY = "-----";

            public PemReader(Reader in) {
                super(in);
            }

            public byte[] readPemObject() {
                try {
                    String line = readLine();
                    while (line != null && !line.startsWith(BEGIN)) {
                        line = readLine();
                    }
                    if (line != null) {
                        line = line.substring(BEGIN.length());
                        int index = line.indexOf('-');
                        if (index > 0 && line.endsWith(END_BOUNDARY) && (line.length() - index) == END_BOUNDARY.length()) {
                            String type = line.substring(0, index);
                            return loadObject(type);
                        }
                    }
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
                return null;
            }

            private byte[] loadObject(String type) {
                String line;
                String endMarker = END + type;
                StringBuilder buf = new StringBuilder();
                try {
                    while ((line = readLine()) != null) {
                        if (line.contains(":")) {
                            continue;
                        }
                        if (line.contains(endMarker)) {
                            break;
                        }
                        buf.append(line.trim());
                    }
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
                if (line == null) {
                    throw new RuntimeException(endMarker + " not found");
                }
                return Base64.getDecoder().decode(buf.toString());
            }
        }

        static class EvidenceHashParam {
            private String fileLabel;
            private List<HashInfo> list;
            static class HashInfo {
                private String filename;
                private String fileHash;

                public String getFilename() {
                    return filename;
                }

                public void setFilename(String filename) {
                    this.filename = filename;
                }

                public String getFileHash() {
                    return fileHash;
                }

                public void setFileHash(String fileHash) {
                    this.fileHash = fileHash;
                }
            }

            public String getFileLabel() {
                return fileLabel;
            }

            public void setFileLabel(String fileLabel) {
                this.fileLabel = fileLabel;
            }

            public List<HashInfo> getList() {
                return list;
            }

            public void setList(List<HashInfo> list) {
                this.list = list;
            }
        }

        static class EvidenceFileParam {
            private String fileLabel;
            private List<Long> files;

            public String getFileLabel() {
                return fileLabel;
            }

            public void setFileLabel(String fileLabel) {
                this.fileLabel = fileLabel;
            }

            public List<Long> getFiles() {
                return files;
            }

            public void setFiles(List<Long> files) {
                this.files = files;
            }
        }

        private String uri = "http://127.0.0.1:18848/api";

        /**
         *
         * @throws Exception
         */
        @Test
        public void detail() throws Exception {
            String apiName = "/evidence/detail";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
            body.put("attestationId","did:bid:efsRrRCTEmA7ZWodWFPkjMW2u5Y4hikv");
            httpRequest.body(JSONUtil.toJsonStr(body));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void list() throws Exception {
            // API path
            String apiName = "/evidence/list";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
            body.put("attestationId","did:bid:efsRrRCTEmA7ZWodWFPkjMW2u5Y4hikv");
            httpRequest.body(JSONUtil.toJsonStr(body));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void hash() throws Exception {
            // API path
            String apiName = "/evidence/hash";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            List<EvidenceHashParam.HashInfo> list = new ArrayList<>();
            EvidenceHashParam.HashInfo hashInfo1 = new EvidenceHashParam.HashInfo();
            hashInfo1.setFilename("test1");
            hashInfo1.setFileHash("98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653");
            list.add(hashInfo1);
            EvidenceHashParam evidenceHashParam = new EvidenceHashParam();
            evidenceHashParam.setFileLabel("标签");
            evidenceHashParam.setList(list);
            httpRequest.body(JSONUtil.toJsonStr(evidenceHashParam));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void file() throws Exception {
            // API path
            String apiName = "/evidence/file";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            List<Long> list = new ArrayList<>();
            list.add(1529663660129480704L);
            EvidenceFileParam evidenceFileParam = new EvidenceFileParam();
            evidenceFileParam.setFileLabel("标签");
            evidenceFileParam.setFiles(list);
            httpRequest.body(JSONUtil.toJsonStr(evidenceFileParam));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }
        @Test
        public void uploadFile() throws Exception {
            // API path
            String apiName = "/file/upload";
            HttpRequest httpRequest = createRequestPost(apiName);
            httpRequest.form("file",new File("/tmp/背景图.png"));
            httpRequest.form("type","pic");

            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }
        @Test
        public void download() throws Exception {
            // API path
            String apiName = "/file/download/1529707935276466176";
            HttpRequest httpRequest = createRequestGet(apiName);

            HttpResponse httpResponse = httpRequest.execute();
            String header = httpResponse.header("Content-Disposition");
            Pattern pattern = Pattern.compile(".*filename=\"(.*)\".*");
            Matcher matcher = pattern.matcher(header);
            String fileName = "";
            if (matcher.matches()) {
                fileName = matcher.group(1);
            }
            byte[] bytes = httpResponse.bodyBytes();
            IoUtil.write(new FileOutputStream("/tmp/" + fileName),true,bytes);
        }

        private HttpRequest createRequestPost(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createPost(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }
        private HttpRequest createRequestGet(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createGet(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }

        private HttpRequest setHttpRequestHeaders(HttpRequest httpRequest) throws Exception {
            // RSA私钥文件路径
            String keyFile = "/tmp/rsa_private.key";
            // 请求头
            String requestId = IdUtil.simpleUUID();
            String accessKey = "9d82aeae8c9b4c479715fc2923619472";
            String nonce = String.valueOf(System.currentTimeMillis() / 1000);

            //待签名数据 = requestId+accessKey+nonce
            String data = requestId + accessKey + nonce;
            // 开始签名
            PrivateKey privateKey = getPrivateKey(new InputStreamReader(new FileInputStream(keyFile)));
            Signature signature = Signature.getInstance("SHA256WithRSA");
            signature.initSign(privateKey);
            signature.update(data.getBytes(StandardCharsets.UTF_8));
            // 签名使用Base64编码后得到的值即为请求头中signature字段的值
            String signatureData = Base64.getEncoder().encodeToString( signature.sign());
            // 构建请求头
            Map<String ,String> headers = new HashMap<>();
            headers.put("request_id", requestId);
            headers.put("access_key", accessKey);
            headers.put("nonce",nonce);
            headers.put("signature",signatureData);
            httpRequest.addHeaders(headers);
            return httpRequest;
        }

        public static PrivateKey getPrivateKey(Reader reader) {
            PemReader pemReader = null;
            try {
                pemReader = new PemReader(reader);
                KeyFactory keyFactory = KeyFactory.getInstance("RSA");
                PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(pemReader.readPemObject());
                return keyFactory.generatePrivate(keySpec);
            } catch (NoSuchAlgorithmException | InvalidKeySpecException e) {
                throw new RuntimeException(e);
            } finally {
                IoUtil.close(pemReader);
            }
        }
    }
