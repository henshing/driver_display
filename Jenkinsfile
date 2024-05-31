pipeline {
    agent any

    environment {
        name = "Starry"
        // 主仓名
        mainRepoName = "Starry"
        // 提交仓名
        currentRepoName = "${GIT_URL.substring(GIT_URL.lastIndexOf('/')+1, GIT_URL.length()-4)}"
        NODE_BASE_NAME = "ui-node-${GIT_COMMIT.substring(0, 6)}"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_sl"
        REPORT_PATH = "allure"
        GITHUB_URL_PREFIX = "https://github.com/henshing/"
        GITHUB_URL_SUFFIX = ".git"
        // 根据内置变量 currentBuild 获取构建号
        buildNumber = "${currentBuild.number}"
        // 构建 Allure 报告地址
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
        REPORT_EMAIL = "1445323887@qq.com"
    }
    
    stages {
        stage("多仓CI") {
            steps {
                script {
                    parallel repoJobs()
                }
            }
        }
        stage("合并报告") {
            steps {
                script {
                    echo "合并所有仓库的 Allure 报告"
                    sh '''
                        rm -rf merged-report
                        mkdir -p merged-report
                        for repo in ${repos().join(' ')}
                        do
                            cp -r $WORKSPACE/$repo/pytest/report/result/* merged-report/
                        done
                        allure generate merged-report -o merged-report/html --clean
                    '''
                }
            }
        }
        stage("展示合并报告") {
            steps {
                allure includeProperties: false, jdk: 'jdk17', report: 'merged-report/html', results: [[path: 'merged-report']]
                echo "Allure 综合报告 URL: ${allureReportUrl}"
            }
        }
    }

    post {
        failure {
            script {
                mail to: "${REPORT_EMAIL}",
                subject: "Pipeline '${JOB_NAME}' (${BUILD_NUMBER}) 执行失败",
                body: "${currentRepoName} 仓 CI 报告链接：\nPipeline '${JOB_NAME}' (${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
        success {
            script {
                mail to: "${REPORT_EMAIL}",
                subject: "Pipeline '${JOB_NAME}' (${BUILD_NUMBER}) 执行成功",
                body: "${currentRepoName} 仓 CI 报告链接：\nPipeline '${JOB_NAME}' (${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
    }
}

def repos() {
    return ["$currentRepoName", "$mainRepoName"]
}

def repoJobs() {
    jobs = [:]
    repos().each { repo ->
        jobs[repo] = {
            stage(repo + "代码检出") {
                echo "$repo 代码检出"
                sh "rm -rf $repo; git clone $GITHUB_URL_PREFIX$repo$GITHUB_URL_SUFFIX; echo `pwd`;"
            }
            stage(repo + "自动化嵌入") {
                echo "$repo pytest嵌入"
                sh "cp -r /home/jenkins_home/pytest $WORKSPACE/$repo"
            }
            stage(repo + "编译测试") {
                withEnv(["repoName=$repo"]) {
                    echo "repoName = ${repoName}"
                    echo "$repo 编译测试"
                    sh 'printenv'
                    echo "--------------------------------------------$repo test start------------------------------------------------"
                    if (repoName == mainRepoName) {
                        sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && python3 -m pytest -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                    } else {
                        sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && python3 -m pytest -sv --alluredir report/result testcase/test_arceos_cargo_test.py --clean-alluredir'
                    }
                    echo "--------------------------------------------$repo test end  ------------------------------------------------"
                }
            }
            stage(repo + "报告生成") {
                echo "$repo 报告生成"
                echo "$repo Allure Report URL: ${allureReportUrl}"
            }
            stage(repo + "结果展示") {
                withEnv(["repoName=$repo"]) {
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
