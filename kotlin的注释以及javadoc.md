# kotlin的注释

	安装插件KDOC,如果android studio中搜索不到插件,进行以下设置
	
	File -> Settings -> Appearance & Behavior -> System settings -> Updates
	Use secure connection (不要勾选这个选项，去掉这个选项前面的√)

## kotlin生成javaDoc

引用dokka包

项目的根build.gradle

    dependencies {
		......
        classpath 'org.jetbrains.dokka:dokka-gradle-plugin:0.10.1'
    }

	
对应moudle的build.gradle

	apply plugin: 'org.jetbrains.dokka'
	android {
		  dokka {
	        // 输出格式，目前支持五种，html, javadoc,html-as-java, markdown,kotlin-website*
	        outputFormat = 'javadoc'
	        // 文档输出目录(app/build/dokka)
	        outputDirectory = "$buildDir/dokka"
	        //配置方式一
	        configuration {
	            // Do not output deprecated members
	            skipDeprecated = true
	            // Emit warnings about not documented members.
	            reportUndocumented = true
	            // Do not create index pages for empty packages
	            skipEmptyPackages = true
	
	            noJdkLink = true
	            noStdlibLink = true
	            noAndroidSdkLink = true
	            //可以屏蔽包
	            perPackageOption {
	                prefix = "x.y.z"
	                suppress = true
	            }
	          
	        }
	
	    }
	}	

然后就可以在右边的gradle的对应moudle的Task中看到documentation

双击即可生成javadoc


 


