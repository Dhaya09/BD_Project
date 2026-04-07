pipeline {
    agent any
    stages {
        stage('Start Cluster') {
            steps {
                bat 'docker start namenode datanode1 datanode2 datanode3'
                bat 'docker exec datanode1 service ssh start'
                bat 'docker exec datanode2 service ssh start'
                bat 'docker exec datanode3 service ssh start'
                bat 'docker exec namenode service ssh start'
                bat 'docker exec namenode /opt/hadoop/sbin/start-dfs.sh'
                bat 'docker exec namenode /opt/hadoop/sbin/start-yarn.sh'
            }
        }
        stage('Prepare HDFS') {
            steps {
                bat 'docker exec namenode hdfs dfs -rm -r -f /user/sales/output'
                bat 'docker exec namenode hdfs dfsadmin -safemode leave'
                bat 'docker exec namenode hdfs dfs -mkdir -p /user/sales/input'
                bat 'docker cp sales_dataset.csv namenode:/opt/results/sales_dataset.csv'
                bat 'docker exec namenode hdfs dfs -put -f /opt/results/sales_dataset.csv /user/sales/input/'
            }
        }
        stage('Run MapReduce') {
            steps {
                bat 'docker exec namenode hadoop jar /opt/mapreduce/SalesAnalysis.jar SalesAnalysis /user/sales/input /user/sales/output/sales'
                bat 'docker exec namenode hadoop jar /opt/mapreduce/StockAnalysis.jar StockAnalysis /user/sales/input /user/sales/output/stock'
            }
        }
        stage('Generate Reports') {
            steps {
                bat 'docker exec namenode python3 /opt/results/generate_sales_stock_csv.py'
                bat 'docker exec namenode python3 /opt/results/generate_sales_report.py'
                bat 'docker exec namenode python3 /opt/results/send_sales_report.py'
            }
        }
        stage('Copy Results to Local') {
            steps {
                bat 'docker cp namenode:/opt/results/sales_final.txt Results/'
                bat 'docker cp namenode:/opt/results/stock_final.txt Results/'
                bat 'docker cp namenode:/opt/results/sales_analysis.csv Results/'
                bat 'docker cp namenode:/opt/results/stock_analysis.csv Results/'
                bat 'docker cp namenode:/opt/results/sales_report.pdf Results/'
            }
        }
        stage('Stop Cluster') {
            steps {
                bat 'docker exec namenode /opt/hadoop/sbin/stop-yarn.sh || exit 0'
                bat 'docker exec namenode /opt/hadoop/sbin/stop-dfs.sh || exit 0'
                bat 'docker stop namenode datanode1 datanode2 datanode3'
            }
        }
    }
    post {
        always {
            bat 'docker exec namenode /opt/hadoop/sbin/stop-yarn.sh || exit 0'
            bat 'docker exec namenode /opt/hadoop/sbin/stop-dfs.sh || exit 0'
            bat 'docker stop namenode datanode1 datanode2 datanode3 || exit 0'
        }
    }
}
