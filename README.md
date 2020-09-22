<div align="center">

## Creating ZIP Files on the Spot


</div>

### Description

This interesting article found on Zend.com will explain to you how to create ZIP files on the fly using PHP.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Christian Mallette](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/christian-mallette.md)
**Level**          |Advanced
**User Rating**    |4.8 (43 globes from 9 users)
**Compatibility**  |PHP 4\.0
**Category**       |[Coding Standards](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/coding-standards__8-33.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/christian-mallette-creating-zip-files-on-the-spot__8-957/archive/master.zip)





### Source Code

<font size="2" face="Arial">Ever wondered how you could create ZIP files on the
fly? In this article, which I found on Zend.com, you'll be able to. The original version can be found here: http://www.zend.com/zend/spotlight/creating-zip-files2.php</font>
<br>
<blockquote>
<p><code><font color="#000000"><font color="#0000bb">&lt;?php<br>
<br>
</font><font color="#ff8000">/*<br>
<br>
Zip file creation class<br>
makes zip files on the fly...<br>
<br>
use the functions add_dir() and add_file() to build the zip file;<br>
see example code below<br>
<br>
by Eric Mueller<br>
http://www.themepark.com<br>
<br>
v1.1 9-20-01<br>
&nbsp;&nbsp;- added comments to example<br>
<br>
v1.0 2-5-01<br>
<br>
initial version with:<br>
&nbsp;&nbsp;- class appearance<br>
&nbsp;&nbsp;- add_file() and file()
methods<br>
&nbsp;&nbsp;- gzcompress()
output hacking<br>
by Denis O.Philippov, webmaster@atlant.ru, http://www.atlant.ru<br>
<br>
*/<br>
<br>
// official ZIP file format: http://www.pkware.com/appnote.txt<br>
<br>
</font><font color="#007700">class </font><font color="#0000bb">zipfile&nbsp;&nbsp;<br>
</font><font color="#007700">{&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;var </font><font color="#0000bb">$datasec </font><font color="#007700">=
array(); </font><font color="#ff8000">// array to store compressed data<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">var </font><font color="#0000bb">$ctrl_dir
</font><font color="#007700">= array(); </font><font color="#ff8000">// central
directory&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">var </font><font color="#0000bb">$eof_ctrl_dir
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x50\x4b\x05\x06\x00\x00\x00\x00&quot;</font><font color="#007700">;
</font><font color="#ff8000">//end of Central directory record<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">var </font><font color="#0000bb">$old_offset
</font><font color="#007700">= </font><font color="#0000bb">0</font><font color="#007700">;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;function </font><font color="#0000bb">add_dir</font><font color="#007700">(</font><font color="#0000bb">$name</font><font color="#007700">)&nbsp;&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">// adds
&quot;directory&quot; to archive - do this before putting any files in
directory!<br>
&nbsp;&nbsp;&nbsp;&nbsp;// $name - name of directory... like this:
&quot;path/&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;// ...then you can add files using add_file with names
like &quot;path/file.txt&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">{&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$name
</font><font color="#007700">= </font><font color="#0000bb">str_replace</font><font color="#007700">(</font><font color="#dd0000">&quot;\\&quot;</font><font color="#007700">,
</font><font color="#dd0000">&quot;/&quot;</font><font color="#007700">, </font><font color="#0000bb">$name</font><font color="#007700">);&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x50\x4b\x03\x04&quot;</font><font color="#007700">;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x0a\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
ver needed to extract<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
gen purpose bit flag<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
compression method<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x00\x00\x00\x00&quot;</font><font color="#007700">;
</font><font color="#ff8000">// last mod time and date<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">// crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$name</font><font color="#007700">)
); </font><font color="#ff8000">//length of pathname<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//extra
field length<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">$name</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
end of &quot;local file header&quot; segment<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// no &quot;file data&quot;
segment for path<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// &quot;data descriptor&quot;
segment (optional but necessary if archive is not served as file)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$crc</font><font color="#007700">);
</font><font color="#ff8000">//crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$c_len</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$unc_len</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// add this entry to array<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">datasec</font><font color="#007700">[]
= </font><font color="#0000bb">$fr</font><font color="#007700">;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$new_offset
</font><font color="#007700">= </font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">implode</font><font color="#007700">(</font><font color="#dd0000">&quot;&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this</font><font color="#007700">-&gt;</font><font color="#0000bb">datasec</font><font color="#007700">));<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
ext. file attributes mirrors MS-DOS directory attr byte, detailed<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// at http://support.microsoft.com/support/kb/articles/Q125/0/19.asp<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// now add to central record<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x50\x4b\x01\x02&quot;</font><font color="#007700">;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
version made by<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x0a\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
version needed to extract<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
gen purpose bit flag<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
compression method<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00\x00\x00&quot;</font><font color="#007700">;
</font><font color="#ff8000">// last mod time &amp; date<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">// crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">0</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$name</font><font color="#007700">)
); </font><font color="#ff8000">//length of filename<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//extra
field length&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//file
comment length<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//disk
number start<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//internal
file attributes<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$ext
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x00\x00\x10\x00&quot;</font><font color="#007700">;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$ext
</font><font color="#007700">= </font><font color="#dd0000">&quot;\xff\xff\xff\xff&quot;</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">16 </font><font color="#007700">); </font><font color="#ff8000">//external
file attributes&nbsp;&nbsp;- 'directory' bit set<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this </font><font color="#007700">-&gt; </font><font color="#0000bb">old_offset
</font><font color="#007700">); </font><font color="#ff8000">//relative offset
of local header<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">old_offset </font><font color="#007700">=
</font><font color="#0000bb">$new_offset</font><font color="#007700">;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">$name</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
optional extra field, file comment goes here<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// save to array<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">ctrl_dir</font><font color="#007700">[]
= </font><font color="#0000bb">$cdrec</font><font color="#007700">;&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;}<br>
<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;function </font><font color="#0000bb">add_file</font><font color="#007700">(</font><font color="#0000bb">$data</font><font color="#007700">,
</font><font color="#0000bb">$name</font><font color="#007700">)&nbsp;&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">// adds &quot;file&quot; to
archive&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;// $data - file contents<br>
&nbsp;&nbsp;&nbsp;&nbsp;// $name - name of file in archive. Add path if your
want<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">{&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$name
</font><font color="#007700">= </font><font color="#0000bb">str_replace</font><font color="#007700">(</font><font color="#dd0000">&quot;\\&quot;</font><font color="#007700">,
</font><font color="#dd0000">&quot;/&quot;</font><font color="#007700">, </font><font color="#0000bb">$name</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//$name
= str_replace(&quot;\\&quot;,
&quot;\\\\&quot;, $name);<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x50\x4b\x03\x04&quot;</font><font color="#007700">;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x14\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
ver needed to extract<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
gen purpose bit flag<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x08\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
compression method<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#dd0000">&quot;\x00\x00\x00\x00&quot;</font><font color="#007700">;
</font><font color="#ff8000">// last mod time and date<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$unc_len
</font><font color="#007700">= </font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$data</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$crc
</font><font color="#007700">= </font><font color="#0000bb">crc32</font><font color="#007700">(</font><font color="#0000bb">$data</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$zdata
</font><font color="#007700">= </font><font color="#0000bb">gzcompress</font><font color="#007700">(</font><font color="#0000bb">$data</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$zdata
</font><font color="#007700">= </font><font color="#0000bb">substr</font><font color="#007700">(
</font><font color="#0000bb">substr</font><font color="#007700">(</font><font color="#0000bb">$zdata</font><font color="#007700">,
</font><font color="#0000bb">0</font><font color="#007700">, </font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$zdata</font><font color="#007700">)
- </font><font color="#0000bb">4</font><font color="#007700">), </font><font color="#0000bb">2</font><font color="#007700">);
</font><font color="#ff8000">// fix crc bug<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$c_len
</font><font color="#007700">= </font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$zdata</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$crc</font><font color="#007700">);
</font><font color="#ff8000">// crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$c_len</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$unc_len</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$name</font><font color="#007700">)
); </font><font color="#ff8000">//length of filename<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//extra
field length<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">$name</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
end of &quot;local file header&quot; segment<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// &quot;file data&quot; segment<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">$zdata</font><font color="#007700">;&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
&quot;data descriptor&quot; segment (optional but necessary if archive is not
served as file)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$crc</font><font color="#007700">);
</font><font color="#ff8000">//crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$c_len</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$fr
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$unc_len</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// add this entry to array<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">datasec</font><font color="#007700">[]
= </font><font color="#0000bb">$fr</font><font color="#007700">;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$new_offset
</font><font color="#007700">= </font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">implode</font><font color="#007700">(</font><font color="#dd0000">&quot;&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this</font><font color="#007700">-&gt;</font><font color="#0000bb">datasec</font><font color="#007700">));<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
now add to central directory record<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">= </font><font color="#dd0000">&quot;\x50\x4b\x01\x02&quot;</font><font color="#007700">;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
version made by<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x14\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
version needed to extract<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
gen purpose bit flag<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x08\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
compression method<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.=</font><font color="#dd0000">&quot;\x00\x00\x00\x00&quot;</font><font color="#007700">;
</font><font color="#ff8000">// last mod time &amp; date<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$crc</font><font color="#007700">);
</font><font color="#ff8000">// crc32<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$c_len</font><font color="#007700">);
</font><font color="#ff8000">//compressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,</font><font color="#0000bb">$unc_len</font><font color="#007700">);
</font><font color="#ff8000">//uncompressed filesize<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$name</font><font color="#007700">)
); </font><font color="#ff8000">//length of filename<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//extra
field length&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//file
comment length<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//disk
number start<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">0 </font><font color="#007700">); </font><font color="#ff8000">//internal
file attributes<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">32 </font><font color="#007700">); </font><font color="#ff8000">//external
file attributes - 'archive' bit set<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this </font><font color="#007700">-&gt; </font><font color="#0000bb">old_offset
</font><font color="#007700">); </font><font color="#ff8000">//relative offset
of local header<br>
//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;echo &quot;old offset is
&quot;.$this-&gt;old_offset.&quot;, new offset is $new_offset&lt;br&gt;&quot;;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">old_offset </font><font color="#007700">=
</font><font color="#0000bb">$new_offset</font><font color="#007700">;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$cdrec
</font><font color="#007700">.= </font><font color="#0000bb">$name</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
optional extra field, file comment goes here<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// save to central directory<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">ctrl_dir</font><font color="#007700">[]
= </font><font color="#0000bb">$cdrec</font><font color="#007700">;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;}<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;function </font><font color="#0000bb">file</font><font color="#007700">()
{ </font><font color="#ff8000">// dump out file&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$data
</font><font color="#007700">= </font><font color="#0000bb">implode</font><font color="#007700">(</font><font color="#dd0000">&quot;&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this </font><font color="#007700">-&gt; </font><font color="#0000bb">datasec</font><font color="#007700">);&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$ctrldir
</font><font color="#007700">= </font><font color="#0000bb">implode</font><font color="#007700">(</font><font color="#dd0000">&quot;&quot;</font><font color="#007700">,
</font><font color="#0000bb">$this </font><font color="#007700">-&gt; </font><font color="#0000bb">ctrl_dir</font><font color="#007700">);&nbsp;&nbsp;<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$data</font><font color="#007700">.&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$ctrldir</font><font color="#007700">.&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">eof_ctrl_dir</font><font color="#007700">.&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">sizeof</font><font color="#007700">(</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">ctrl_dir</font><font color="#007700">)).&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
total # of entries &quot;on this disk&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;v&quot;</font><font color="#007700">,
</font><font color="#0000bb">sizeof</font><font color="#007700">(</font><font color="#0000bb">$this
</font><font color="#007700">-&gt; </font><font color="#0000bb">ctrl_dir</font><font color="#007700">)).&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
total # of entries overall<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$ctrldir</font><font color="#007700">)).&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
size of central dir<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#0000bb">pack</font><font color="#007700">(</font><font color="#dd0000">&quot;V&quot;</font><font color="#007700">,
</font><font color="#0000bb">strlen</font><font color="#007700">(</font><font color="#0000bb">$data</font><font color="#007700">)).&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
offset to start of central dir<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#dd0000">&quot;\x00\x00&quot;</font><font color="#007700">;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#ff8000">//
.zip file comment length<br>
&nbsp;&nbsp;&nbsp;&nbsp;</font><font color="#007700">}<br>
}&nbsp;&nbsp;<br>
<br>
</font><font color="#0000bb">?&gt;</font>
</font></code></p>
</blockquote>
<p><code><font size="2" face="Arial" color="#000000"><b>Example Usage of the
Class</b></font></code></p>
<blockquote>
 <p><code><font color="#000000"><font color="#0000bb">&lt;?php<br>
 <br>
 $zipfile </font><font color="#007700">= new </font><font color="#0000bb">zipfile</font><font color="#007700">();&nbsp;&nbsp;<br>
 <br>
 </font><font color="#ff8000">// add the subdirectory ... important!<br>
 </font><font color="#0000bb">$zipfile </font><font color="#007700">-&gt; </font><font color="#0000bb">add_dir</font><font color="#007700">(</font><font color="#dd0000">&quot;dir/&quot;</font><font color="#007700">);<br>
 <br>
 </font><font color="#ff8000">// add the binary data stored in the string 'filedata'<br>
 </font><font color="#0000bb">$filedata </font><font color="#007700">= </font><font color="#dd0000">&quot;(read
 your file into $filedata)&quot;</font><font color="#007700">;&nbsp;&nbsp;<br>
 </font><font color="#0000bb">$zipfile </font><font color="#007700">-&gt; </font><font color="#0000bb">add_file</font><font color="#007700">(</font><font color="#0000bb">$filedata</font><font color="#007700">,
 </font><font color="#dd0000">&quot;dir/file.txt&quot;</font><font color="#007700">);&nbsp;&nbsp;<br>
 <br>
 </font><font color="#ff8000">// the next three lines force an immediate
 download of the zip file:<br>
 </font><font color="#0000bb">header</font><font color="#007700">(</font><font color="#dd0000">&quot;Content-type:
 application/octet-stream&quot;</font><font color="#007700">);&nbsp;&nbsp;<br>
 </font><font color="#0000bb">header</font><font color="#007700">(</font><font color="#dd0000">&quot;Content-disposition:
 attachment; filename=test.zip&quot;</font><font color="#007700">);&nbsp;&nbsp;<br>
 echo </font><font color="#0000bb">$zipfile </font><font color="#007700">-&gt; </font><font color="#0000bb">file</font><font color="#007700">();&nbsp;&nbsp;<br>
 <br>
 <br>
 </font><font color="#ff8000">// OR instead of doing that, you can write out
 the file to the loca disk like this:<br>
 </font><font color="#0000bb">$filename </font><font color="#007700">= </font><font color="#dd0000">&quot;output.zip&quot;</font><font color="#007700">;<br>
 </font><font color="#0000bb">$fd </font><font color="#007700">= </font><font color="#0000bb">fopen
 </font><font color="#007700">(</font><font color="#0000bb">$filename</font><font color="#007700">,
 </font><font color="#dd0000">&quot;wb&quot;</font><font color="#007700">);<br>
 </font><font color="#0000bb">$out </font><font color="#007700">= </font><font color="#0000bb">fwrite
 </font><font color="#007700">(</font><font color="#0000bb">$fd</font><font color="#007700">,
 </font><font color="#0000bb">$zipfile </font><font color="#007700">-&gt; </font><font color="#0000bb">file</font><font color="#007700">());<br>
 </font><font color="#0000bb">fclose </font><font color="#007700">(</font><font color="#0000bb">$fd</font><font color="#007700">);<br>
 <br>
 </font><font color="#ff8000">// then offer it to the user to download:<br>
 </font><font color="#007700">&lt;</font><font color="#0000bb">a href</font><font color="#007700">=</font><font color="#dd0000">&quot;output.zip&quot;</font><font color="#007700">&gt;</font><font color="#0000bb">Click
 here to download the </font><font color="#007700">new </font><font color="#0000bb">zip
 file</font><font color="#007700">.&lt;/</font><font color="#0000bb">a</font><font color="#007700">&gt;<br>
 <br>
 </font><font color="#0000bb">?&gt;</font></font></code></p>
 <p>&nbsp;</p>
</blockquote>

