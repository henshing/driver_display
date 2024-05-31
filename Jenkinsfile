pipeline {
    agent any

    environment {
        name = "Starry"
        mainRepoName = "Starry"
        // repoName = "Starry"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_sl"
        REPORT_PATH = "allure"
        GITHUB_URL_PREFIX = "https://github.com/henshing/"
        GITHUB_URL_SUFFIX = ".git"
        //根据内置变量currentBuild获取构建号
        buildNumber = "${currentBuild.number}"
        // 构建 Allure 报告地址
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
    }
    
    stages {
        stage("多仓CI") {
            steps {
                script {
                    parallel repoJobs()
                }
            }
        }
    }
}

def repos() {
  // return ["Starry", "driver_display",  "axtrap"]
  return ["driver_display",  "axtrap"]
}

def repoJobs() {
  jobs = [:]
  repos().each { repo ->
    jobs[repo] = { 
        stage(repo) {
           echo "Step for $repo"
        }
        stage(repo + "代码检出"){
           echo "$repo 代码检出"
           sh "rm -rf  $repo; git clone $GITHUB_URL_PREFIX$repo$GITHUB_URL_SUFFIX; echo `pwd`;"
        }
        stage(repo + "自动化嵌入"){
            echo "$repo pytest嵌入"
            // sh 'echo $PATH'
            // sh 'printenv'
            sh "cp -r /home/jenkins_home/pytest $WORKSPACE/$repo"
        }
        stage(repo + "编译测试"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 编译测试"
                sh 'printenv'
                echo "--------------------------------------------$repo test start------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && python3 -m pytest -sv --alluredir report/result testcase/test_arceos_cargo_test.py --clean-alluredir'
                echo "--------------------------------------------$repo test end  ------------------------------------------------"
            }
        }
        stage(repo + "报告生成") {
            echo "$repo 报告生成"
            // 输出 Allure 报告地址
            echo "$repo Allure Report URL: ${allureReportUrl}"
        }
        stage(repo + "结果展示"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 结果展示"
                sh 'printenv'
                echo "-------------------------$repo allure report generating start---------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                allure includeProperties: false, jdk: 'jdk17', report: "$repo/pytest/report/html", results: [[path: "$repo/pytest/report/result"]]
                echo "-------------------------$repo allure report generating end ----------------------------------------------------"
            }
        }
    }
  }
  return jobs
}
