---
layout: post
title: "Java备份恢复MySQL数据库"
date: 2015-04-19 18:04:20 +0800
comments: true
categories: 
---
　　项目中需要在数据清算前对数据库进行备份，清算产生问题时对数据库进行恢复，要在程序里实现。

　　一般备份MySQL用dump就可以了，如果需要在Java程序里实现，应该是调用系统命令就可以，也就是说用java里的`Runtime`就可以实现。

####备份

#####第一版代码

    Runtime rt = Runtime.getRuntime();
    Process pro = rt.exec("mysqldump -uUSER -pPWD DATABASE> PATH/file.sql");

但是结果报错了。

其中有了两个问题：

1. mysqldump不一定在环境变量中，即使在，有的时候这样执行也是执行不了的。
2. 这种命令执行方式java不能识别重定向符 >。

修改如下：

        String time = new SimpleDateFormat("yyyyMMddHHmmss").format(new Date()).toString();
        Process pro = rt.exec("mysqldump -uUSER -pPWD --default-character=gbk -rPATH" + time + ".sql DATABASE");
        BufferedReader br = new BufferedReader(new InputStreamReader(pro.getErrorStream()));
        String errorLine = null;
        while ((errorLine = br.readLine()) != null) {
            System.out.println(errorLine);
        }
        br.close();  

其中`exec`的参数还可以写成字符串数组，能够解决不识别`mysqldump`的问题。另外，`Process`的waitFor方法会在执行后返回int值，如果为0则执行正常。

    @Test
    public void backupDB() throws Exception {
        Runtime rt = Runtime.getRuntime();
        Process pro = rt.exec(getCommand());
        BufferedReader br = new BufferedReader(new InputStreamReader(pro.getErrorStream()));
        String errorLine = null;
        while ((errorLine = br.readLine()) != null) {
            System.out.println(errorLine);
        }
        br.close();
        int result = pro.waitFor();
        if (result != 0) {
            throw new Exception("数据库备份失败！ ");
        }
    }  


    private String[] getCommand() {
        String[] cmd = new String[3];
        String time = new SimpleDateFormat("yyyyMMddHHmmss").format(new Date()).toString();
        String os = System.getProperties().getProperty("os.name");
        if (os.startsWith("Win")) {
            cmd[0] = "cmd.exe";
            cmd[1] = "/c";
        } else {
            cmd[0] = "/bin/sh";
            cmd[1] = "-c";
        }
        StringBuilder arg = new StringBuilder();
        arg.append("mysqldump ");
        arg.append("-uUSR ");
        arg.append("-pPWD ");
        arg.append("--default-character=gbk ");
        arg.append("--skip-opt ");
        arg.append("--add-drop-database ");
        arg.append("--routines ");
        arg.append("--triggers ");
        arg.append("--compress ");
        arg.append("-r");
        arg.append("PATH");
        arg.append(time);
        arg.append(".sql ");
        arg.append("--databases DATABASE");
        cmd[2] = arg.toString();
        return cmd;
    }  

####说明

--default-character=gbk    指定字符集

--skip-opt   禁用将结果存入内存，否则大表会出现问题

--add-drop-database   在CREATE DATABASE之前加 drop语句

--routines   转储数据库中的函数和程序

--compress   压缩传输的信息

###恢复

恢复是用`mysql`命令，方式和备份很像

    @Test
    public void restoreDB() throws Exception {
        Runtime rt = Runtime.getRuntime();
        Process pro = rt.exec(getCommand());
        BufferedReader br = new BufferedReader(new InputStreamReader(pro.getErrorStream()));
        String errorLine = null;
        while ((errorLine = br.readLine()) != null) {
            System.out.println(errorLine);
        }
        br.close();
        int result = pro.waitFor();
        if (result != 0) {
            throw new Exception("数据库恢复失败！ ");
        }
    }
    private String[] getCommand() {
        String[] cmd = new String[3];
        String os = System.getProperties().getProperty("os.name");
        if (os.startsWith("Win")) {
            cmd[0] = "cmd.exe";
            cmd[1] = "/c";
        } else {
            cmd[0] = "/bin/sh";
            cmd[1] = "-c";
        }
        StringBuilder arg = new StringBuilder();
        arg.append("mysql ");
        arg.append("-uUSR ");
        arg.append("-pPWD ");
        arg.append("< ");
        arg.append(getFile());
        cmd[2] = arg.toString();
        return cmd;
    }
    private String getFile() {
        File dir = new File("PATH");
        String[] files = dir.list(new FilenameFilter() {
            @Override
            public boolean accept(File dir, String name) {
                return name.endsWith(".sql");
            }
        });
        return Collections.max(Arrays.asList(files));
    }  