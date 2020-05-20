# 依赖管理

 如果使用了第三方依赖关系，则可以在提交作业时使用以下Python Table API或直接通过[命令行参数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/cli.html#usage)指定依赖关系。

<table>
  <thead>
    <tr>
      <th style="text-align:left">APIs</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>add_python_file(file_path)</b>
      </td>
      <td style="text-align:left">&#x6DFB;&#x52A0;python&#x6587;&#x4EF6;&#x4F9D;&#x8D56;&#x6027;&#xFF0C;&#x53EF;&#x4EE5;&#x662F;python&#x6587;&#x4EF6;&#xFF0C;python&#x8F6F;&#x4EF6;&#x5305;&#x6216;&#x672C;&#x5730;&#x76EE;&#x5F55;&#x3002;&#x5B83;&#x4EEC;&#x5C06;&#x88AB;&#x6DFB;&#x52A0;&#x5230;python
        UDF worker&#x7684;PYTHONPATH&#x4E2D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>set_python_requirements(requirements_file_path, requirements_cache_dir=None)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x4E00;&#x4E2A;require.txt&#x6587;&#x4EF6;&#xFF0C;&#x8BE5;&#x6587;&#x4EF6;&#x5B9A;&#x4E49;&#x4E86;&#x7B2C;&#x4E09;&#x65B9;&#x4F9D;&#x8D56;&#x9879;&#x3002;&#x8FD9;&#x4E9B;&#x4F9D;&#x8D56;&#x9879;&#x5C06;&#x5B89;&#x88C5;&#x5230;&#x4E00;&#x4E2A;&#x4E34;&#x65F6;&#x76EE;&#x5F55;&#x4E2D;&#xFF0C;&#x5E76;&#x6DFB;&#x52A0;&#x5230;python
          UDF worker&#x7684;PYTHONPATH&#x4E2D;&#x3002;&#x5BF9;&#x4E8E;&#x7FA4;&#x96C6;&#x4E2D;&#x65E0;&#x6CD5;&#x8BBF;&#x95EE;&#x7684;&#x4F9D;&#x8D56;&#x9879;&#xFF0C;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x53C2;&#x6570;&#x201C;
          requirements_cached_dir&#x201D;&#x6307;&#x5B9A;&#x5305;&#x542B;&#x8FD9;&#x4E9B;&#x4F9D;&#x8D56;&#x9879;&#x7684;&#x5B89;&#x88C5;&#x5305;&#x7684;&#x76EE;&#x5F55;&#x3002;&#x5B83;&#x5C06;&#x88AB;&#x4E0A;&#x4F20;&#x5230;&#x96C6;&#x7FA4;&#x4EE5;&#x652F;&#x6301;&#x79BB;&#x7EBF;&#x5B89;&#x88C5;&#x3002;
          <br
          />
        </p>
        <p>&#x8BF7;&#x786E;&#x4FDD;&#x5B89;&#x88C5;&#x5305;&#x4E0E;&#x7FA4;&#x96C6;&#x7684;&#x5E73;&#x53F0;&#x548C;&#x6240;&#x4F7F;&#x7528;&#x7684;python&#x7248;&#x672C;&#x5339;&#x914D;&#x3002;&#x8FD9;&#x4E9B;&#x8F6F;&#x4EF6;&#x5305;&#x5C06;&#x4F7F;&#x7528;pip&#x5B89;&#x88C5;&#xFF0C;&#x56E0;&#x6B64;&#x8FD8;&#x8981;&#x786E;&#x4FDD;Pip&#x7684;&#x7248;&#x672C;&#xFF08;&#x7248;&#x672C;&gt;
          = 7.1.0&#xFF09;&#x548C;SetupTools&#x7684;&#x7248;&#x672C;&#xFF08;&#x7248;&#x672C;&gt;
          = 37.0.0&#xFF09;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>add_python_archive(archive_path, target_dir=None)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;python&#x5B58;&#x6863;&#x6587;&#x4EF6;&#x4F9D;&#x8D56;&#x9879;&#x3002;&#x8BE5;&#x6587;&#x4EF6;&#x5C06;&#x88AB;&#x63D0;&#x53D6;&#x5230;python
          UDF worker&#x7684;&#x5DE5;&#x4F5C;&#x76EE;&#x5F55;&#x4E2D;&#x3002;&#x5982;&#x679C;&#x6307;&#x5B9A;&#x4E86;&#x53C2;&#x6570;&#x201C;
          target_dir&#x201D;&#xFF0C;&#x5219;&#x5B58;&#x6863;&#x6587;&#x4EF6;&#x5C06;&#x88AB;&#x63D0;&#x53D6;&#x5230;&#x540D;&#x4E3A;&#x201C;
          target_dir&#x201D;&#x7684;&#x76EE;&#x5F55;&#x4E2D;&#x3002;&#x5426;&#x5219;&#xFF0C;&#x5B58;&#x6863;&#x6587;&#x4EF6;&#x5C06;&#x88AB;&#x63D0;&#x53D6;&#x5230;&#x4E0E;&#x5B58;&#x6863;&#x6587;&#x4EF6;&#x540C;&#x540D;&#x7684;&#x76EE;&#x5F55;&#x4E2D;&#x3002;
          <br
          />
        </p>
        <p>&#x8BF7;&#x786E;&#x4FDD;&#x4E0A;&#x8F7D;&#x7684;python&#x73AF;&#x5883;&#x4E0E;&#x96C6;&#x7FA4;&#x8FD0;&#x884C;&#x6240;&#x5728;&#x7684;&#x5E73;&#x53F0;&#x5339;&#x914D;&#x3002;&#x76EE;&#x524D;&#x4EC5;&#x652F;&#x6301;zip&#x683C;&#x5F0F;&#x3002;&#x5373;zip&#xFF0C;jar&#xFF0C;whl&#xFF0C;egg&#x7B49;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>set_python_executable(python_exec)</b>
      </td>
      <td style="text-align:left">
        <p>&#x8BBE;&#x7F6E;&#x7528;&#x4E8E;&#x6267;&#x884C;python udf&#x5DE5;&#x4F5C;&#x7A0B;&#x5E8F;&#x7684;python&#x89E3;&#x91CA;&#x5668;&#x7684;&#x8DEF;&#x5F84;&#xFF0C;&#x4F8B;&#x5982;
          &quot;/usr/local/bin/python3&quot;.
          <br />
        </p>
        <p>&#x8BF7;&#x786E;&#x4FDD;&#x6307;&#x5B9A;&#x7684;&#x73AF;&#x5883;&#x4E0E;&#x8FD0;&#x884C;&#x7FA4;&#x96C6;&#x7684;&#x5E73;&#x53F0;&#x5339;&#x914D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>