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
                bat 'ping -n 15 127.0.0.1 > nul'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfsadmin -safemode leave'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -rm -r -f /user/sales/output'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -mkdir -p /user/sales/input'
                bat 'docker cp sales_dataset.csv namenode:/opt/results/sales_dataset.csv'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -put -f /opt/results/sales_dataset.csv /user/sales/input/'
            }
        }
        stage('Run MapReduce') {
            steps {
                bat 'docker exec namenode /opt/hadoop/bin/hadoop jar /opt/mapreduce/SalesAnalysis.jar SalesAnalysis /user/sales/input /user/sales/output/sales'
                bat 'docker exec namenode /opt/hadoop/bin/hadoop jar /opt/mapreduce/StockAnalysis.jar StockAnalysis /user/sales/input /user/sales/output/stock'
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
                bat 'mkdir C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results || exit 0'
                bat 'mkdir C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\sales_output || exit 0'
                bat 'mkdir C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\stock_output || exit 0'

                // Final combined outputs
                bat 'docker cp namenode:/opt/results/sales_final.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\'
                bat 'docker cp namenode:/opt/results/stock_final.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\'
                bat 'docker cp namenode:/opt/results/sales_analysis.csv C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\'
                bat 'docker cp namenode:/opt/results/stock_analysis.csv C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\'
                bat 'docker cp namenode:/opt/results/sales_report.pdf C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\'

                // Raw MapReduce part files
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/sales/output/sales /opt/results/sales_output'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/sales/output/stock /opt/results/stock_output'
                bat 'docker cp namenode:/opt/results/sales_output C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\sales_output\\'
                bat 'docker cp namenode:/opt/results/stock_output C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\BD_Dataset\\Results\\stock_output\\'
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
