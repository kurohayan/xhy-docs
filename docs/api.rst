接口
===============

初始化请求参数
------------------
::

    // 构建请求
    HttpRequest httpRequest = HttpUtil.createPost(uri + apiName);
    // 构建请求头
    Map<String ,String> headers = new HashMap<>();
    headers.put("request_id", requestId);
    headers.put("access_key", accessKey);
    headers.put("nonce",nonce);
    headers.put("signature",signatureData);
    httpRequest.addHeaders(headers);

存证详情 - /evidence/detail
----------------------

查询存证详情。

json
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
attestationId       存证id                                  必选
=================  ======================================= ================

返回的data
^^^^^^^^^^^^^^

调用存证详情成功后会返回详情数据

=======================  ================================
字段名 				        描述
=======================  ================================
attestationId               存证id
evidenceShareCode           证据提取码
pdfFileKey                  pdf文件id
fileHash                    存证文件hash
dataExpireTime              存证文件过期时间
attestationType             存证类型  1.文件 2.hash
dataExpireFlag              存证文件是否已过期
userId                      用户id
fileLabel                   文件标签
auditTime                   审核时间
auditResult                 审核结果
filename                    文件名
createTime                  创建时间
upChainTime                 上链时间
attestationChannel          数据来源  1.自助 2.API
dataFileKey                 存证文件的文件id
username                    用户名称
checkBean                   链信息
checkBean.blockHash         链hash
checkBean.fileName          文件名称
checkBean.evidenceTime      存证时间
checkBean.flag              是否上链
checkBean.attestationId     存证id
checkBean.confirmTime       区块创建时间
checkBean.confirmHash       区块hash
checkBean.ledgerSeq         区块高度
checkBean.hash              文件hash
=======================  ================================


以java为例::

	// 构建请求参数
    Map<String ,Object> body = new HashMap<>();
    body.put("attestationId","did:bid:efsRrRCTEmA7ZWodWFPkjMW2u5Y4hikv");
    httpRequest.body(JSONUtil.toJsonStr(body));
    HttpResponse httpResponse = httpRequest.execute();
    String result = httpResponse.body();

存证列表 - /evidence/list
----------------------

获取存证列表

json
^^^^^^^^^^^^^^^

=================  ================================================ ============
参数名 				描述                                              是否可选
=================  ================================================ ============
evidenceType        存证类型 1.文件存证  2.hash存证                       非必选
evidenceChannel     存证方式 1.自助  2.API                               非必选
state               3.待支付 4.上链中 5.存证成功 6.存证失败 7.安全审核中      非必选
startTime           开始时间                                             非必选
endTime             结束时间                                             非必选
pageNumber          当前页码                                             非必选
pageSize            每页显示数量 最大50                                    非必选
filename            文件名称                                             非必选
=================  ================================================ ============


返回的data
^^^^^^^^^^^^^^

调用存证获取列表接口成功后会返回存证列表

=====================  ===========================================================
字段名 				    描述
=====================  ===========================================================
totalPage               当前页
pageSize                每页显示数量
pageNum                 总页数
rows                    存证数据对象info
info.evidenceChannel    存证方式 1.自助  2.API
info.attestationId      存证id
info.auditTime          审核时间
info.auditResult        审核结果
info.fileHash           文件hash
info.userId             用户id
info.fileLabel          文件标签
info.filename           文件名
info.fileSize           文件大小
info.createTime         创建时间
info.upChainTime        上链时间
info.evidenceType       存证类型 1:文件存证,  2:hash存证
info.state              1.待审核 2.待复审 3.待支付 4.上链中 5.存证成功 6.存证失败
info.username           用户名称
=====================  ===========================================================


以java为例::

    // 构建请求参数
    Map<String ,Object> body = new HashMap<>();
    body.put("evidenceType",1);
    httpRequest.body(JSONUtil.toJsonStr(body));
    HttpResponse httpResponse = httpRequest.execute();
    String result = httpResponse.body();

hash存证（sha256） - /evidence/hash
------------------------------------
用户进行hash存证。

json
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
fileLabel           文件标签                                     必选
list                HashInfo对象列表                            必选
HashInfo.filename   文件名                                     必选
HashInfo.fileHash   文件hash                                   必选
=================  ======================================= ================

返回的data
^^^^^^^^^^^^^^

调用hash存证接口成功后会返回存证id列表

===================  ================================
字段名 				    描述
===================  ================================
list                    bean对象列表
bean.hash               文件hash
bean.attestationId      存证id
===================  ================================

以java为例::

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

文件存证 - /evidence/file
------------------------------------
用户进行文件存证

json
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 			       描述                                     是否可选
=================  ======================================= ================
fileLabel           文件标签                                     必选
files                文件id列表                                 必选
files[0]             文件id                                    必选
=================  ======================================= ================



返回的data
^^^^^^^^^^^^^^

调用文件接口成功后会返回文件id对应的存证id

===================  ================================
字段名 				    描述
===================  ================================
list                    bean对象列表
bean.id                 文件id
bean.attestationId      存证id
===================  ================================

以java为例::

    // 构建请求参数
    List<Long> list = new ArrayList<>();
    list.add(1529663660129480704L);
    EvidenceFileParam evidenceFileParam = new EvidenceFileParam();
    evidenceFileParam.setFileLabel("标签");
    evidenceFileParam.setFiles(list);
    httpRequest.body(JSONUtil.toJsonStr(evidenceFileParam));
    HttpResponse httpResponse = httpRequest.execute();
    String result = httpResponse.body();


上传文件 - /file/upload
-------------------------------

客户可以通过该接口上传文件并获取文件id，文件未存证时会有使用期限。

form-data
^^^^^^^^^^^^^^^

=========  ================================================= ==============
参数名         描述                                                是否可选
=========  ================================================= ==============
file         文件                                                 必选
type         文件类型 doc.文档 pic.图片 audio.音频 video.视频        必选
=========  ================================================= ==============

返回的data
^^^^^^^^^^^^^^

=================  ========================
字段名 				描述
=================  ========================
fileKey                 文件id
=================  ========================


下载存证或pdf文件 - /file/download/{fileKey}
--------------------------------------------------------------

存证原文件或pdf下载

Path
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                      是否可选
=================  ======================================= ================
fileKey                文件id                                必选
=================  ======================================= ================

返回的文件
^^^^^^^^^^^^^^^

该接口会返回存证文件以及文件名，文件就是http返回结果的body，文件名存放在http的header中，header的名称是Content-Disposition，header值形如::

	form-data; name=Content-Disposition; filename=5Yhus2mVSMnQRXobRJCYgt.zip

以java为例::

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

