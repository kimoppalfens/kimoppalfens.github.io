<p>Hi All,</p>
<p>A growing number of customers is contacting us about the issue going on below on their Windows 7 32 bit machines. I don't often ask people to distribute my blog information further. But quite a few customers should probably be warned for this issue.</p>
<p><strong>Problem description</strong></p>
<p>An issues exists at present where Windows 7 32 bit machines will reply compliant/installed on any software update they scan for, even the ones that aren't installed.</p>
<p>I have customers reporting updates failing to install because of this, and one where Cumulative Updates for ConfigMgr started reporting compliant without them creating a deployment for the updated client.</p>
<p>The problem can be seen at the client side in the Windowsupdate.log. When your log contains the following text "</p>
<p><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;">GetWARNING: ISusInternal::GetUpdateMetadata2 failed, hr=8007000E </span></span></span>"</p>
<p>You're probably another victim of this terrible issue. The concern here is that a lot of environments might be unaware they have this issue, as nothing will point it out when looking at things centrally from the Admin UI. Clients will just report compliant on all their software update deployments.</p>
<p><strong>Identifying the problem in your environment</strong></p>
<p>The easiest way I could come up with to identify this problem in your environment is to create a configuration item to detect it. To do this:</p>
<ol>
<li>create a script configuration item.</li>
<li>Select All Windows 7 32 bit as the supported platform</li>
<li>Use String as the data type</li>
<li>Choose powershell as your script language of choice</li>
<li>Paste the following text in the discovery script:<span style="color: #0000ff; font-family: Lucida Console; font-size: xx-small;"><span style="color: #0000ff; font-family: Lucida Console; font-size: xx-small;"><span style="color: #0000ff; font-family: Lucida Console; font-size: xx-small;">select-string</span></span></span><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;"><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;"><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;">-pattern</span></span></span><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;">'GetWARNING: ISusInternal::GetUpdateMetadata2 failed, hr=8007000E'</span></span></span><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;"><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;"><span style="color: #000080; font-family: Lucida Console; font-size: xx-small;">-path</span></span></span><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;">"</span></span></span><span style="color: #ff4500; font-family: Lucida Console; font-size: xx-small;"><span style="color: #ff4500; font-family: Lucida Console; font-size: xx-small;"><span style="color: #ff4500; font-family: Lucida Console; font-size: xx-small;">$env:windir</span></span></span><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;"><span style="color: #8b0000; font-family: Lucida Console; font-size: xx-small;">\windowsupdate.log" </span></span></span></li>
<li>Add the configuration item to a Configuration baseline</li>
<li>Deploy the configuration baseline to All Windows 7 32bit machines</li>
<li>The report list of assets by compliance state for a given baseline is a good report to check the results.</li>
<li>!!!! <strong>Any machines reporting compliant to this baseline have a serious issue as they won't install any software updates, yet report compliant on all !!!!</strong></li>
</ol>
<p>&nbsp;</p>
<p><strong>Possible workarounds</strong></p>
<ol>
<ol>
<li>Decline unneeded updates within the WSUS server (Declined updates do not get offered to clients during scans.)
<ol>
<li>Unneeded updates include superseded updates, updates for products and/or classifications that are not present in the client environment, and expired updates.</li>
<li>You can manually decline the updates within the WSUS console or use a script method . <strong>NOTE:  Always backup the WSUS database (SUSDB) prior to performing any changes like this.</strong></li>
<li>After declining unneeded updates, re-index the susdb, and run WSUS Server Cleanup Wizard:<a href="https://gallery.technet.microsoft.com/ScriptCenter/6f8cde49-5c52-4abd-9820-f1d270ddea61/">https://gallery.technet.microsoft.com/ScriptCenter/6f8cde49-5c52-4abd-9820-f1d270ddea61/</a><a href="https://technet.microsoft.com/en-us/library/dd939856(v=ws.10).aspx">https://technet.microsoft.com/en-us/library/dd939856(v=ws.10).aspx</a></li>
</ol>
</li>
</ol>
</ol>
<ul>
<li><b><b>Set user VA to 3072 MB: <strong>bcdedit /set IncreaseUserVA 3072 </strong></b></b>
<ol>
<li>This will free up another GB of memory in user space..</li>
<li>This does require a restart of the machine.</li>
<li>It’s possible some machines or applications may have problems when this setting is enabled</li>
</ol>
<p>&nbsp;</li>
</ul>
<p>&nbsp;</p>
<ol>
<li>Move wuauserv to its own SVCHost instance running following commands in elevated command prompt:
<ol>
<li>Net stop wuauserv</li>
<li>‘sc config wuauserv type= own’</li>
<li>Net start wuauserv</li>
</ol>
</li>
</ol>
<p><strong>More details:</strong></p>
<p>You can find the nitty gritty details and soulmates in this forum post.</p>
<p>https://social.technet.microsoft.com/Forums/en-US/cf8fbe28-714d-49d3-b2ce-5cc5f6f79c63/some-clients-not-updating-reporting-compliant-hr8007000e-error-in-windowsupdatelog?forum=configmanagersecurity</p>
<p>Technorati Tags: SystemCenter<br />
Tags van Technorati: SCCM,ConfigMGr</p>
<p>Enjoy.<br />
"The M in WMI stands for Magic"<br />
""Everyone is an expert at someting" Kim Oppalfens - ConfigMgr Expert for lack of any other expertise<br />
System Center Configuration Manager MVP<br />
http://www.scug.be/blogs/sccm/default.aspx</p>
<p>http://www.linkedin.com/in/kimoppalfens</p>
<p>http://twitter.com/thewmiguy</p>
