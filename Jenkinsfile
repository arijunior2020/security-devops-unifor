pipeline {
    agent any
    environment {
        DEFECTDOJO_URL = "https://9cdb-168-196-22-16.ngrok-free.app" // URL do DefectDojo
        DEFECTDOJO_TOKEN = "28ff908b602cd7716ead574ce5a0872d4a2390e1" // Token do DefectDojo
        ENGAGEMENT_ID = "1" // ID do engajamento
    }
    stages {
        stage('Configurar Diretório como Seguro no Git') {
            steps {
                sh '''
                echo "Configurando diretório como seguro para o Git..."
                git config --global --add safe.directory /var/jenkins_home/workspace/Pipeline_VAmPI
                '''
            }
        }
        stage('Clone ou Atualizar Repositório') {
            steps {
                sh '''
                if [ -d "/var/jenkins_home/workspace/Pipeline_VAmPI/.git" ]; then
                    echo "Repositório já existe. Atualizando..."
                    cd /var/jenkins_home/workspace/Pipeline_VAmPI
                    git reset --hard
                    git clean -fd
                    git pull origin master
                else
                    echo "Clonando o repositório..."
                    git clone https://github.com/erev0s/VAmPI.git /var/jenkins_home/workspace/Pipeline_VAmPI
                fi
                '''
            }
        }
        stage('Análise SAST - Semgrep') {
            steps {
                script {
                    sh '''
                    echo "Executando análise com Semgrep..."
                    docker exec tools semgrep --config=auto /var/jenkins_home/workspace/Pipeline_VAmPI -o /tools/semgrep.json --json || {
                        echo "Erro ao executar Semgrep.";
                        exit 1;
                    }

                    echo "Validando se o relatório Semgrep foi gerado..."
                    if ! docker exec tools test -s /tools/semgrep.json; then
                        echo "Erro: Relatório Semgrep não foi gerado ou está vazio.";
                        exit 1;
                    fi

                    echo "Enviando relatório Semgrep para DefectDojo..."
                    docker exec tools curl -X POST -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
                        -F "scan_type=Semgrep JSON Report" \
                        -F "engagement=${ENGAGEMENT_ID}" \
                        -F "file=@/tools/semgrep.json" \
                        -F "verified=false" \
                        -F "active=true" \
                        ${DEFECTDOJO_URL}/api/v2/import-scan/ || {
                        echo "Erro ao enviar relatório Semgrep para DefectDojo.";
                        exit 1;
                    }

                    echo "Relatório Semgrep Enviado com Sucesso!"
                    '''
                }
            }
        }
        stage('Análise SAST - Bandit') {
            steps {
                script {
                    sh '''
                    echo "Executando análise com Bandit..."
                    docker exec tools bandit -r /var/jenkins_home/workspace/Pipeline_VAmPI -o /tools/bandit.json -f json || {
                        echo "Erro ao executar Bandit.";
                    }
        
                    echo "Validando se o relatório Bandit foi gerado..."
                    if ! docker exec tools test -s /tools/bandit.json; then
                        echo "Erro: Relatório Bandit não foi gerado ou está vazio.";
                        exit 1;
                    fi
        
                    echo "Enviando relatório Bandit para DefectDojo..."
                    docker exec tools curl -X POST -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
                        -F "scan_type=Bandit Scan" \
                        -F "engagement=${ENGAGEMENT_ID}" \
                        -F "file=@/tools/bandit.json" \
                        -F "verified=false" \
                        -F "active=true" \
                        ${DEFECTDOJO_URL}/api/v2/import-scan/ || {
                        echo "Erro ao enviar relatório Bandit para DefectDojo.";
                        exit 1;
                    }
        
                    echo "Relatório Bandit Enviado com Sucesso!"
                    '''
                }
            }
        }

        stage('Análise SCA - Trivy') {
            steps {
                script {
                    sh '''
                    echo "Executando análise com Trivy..."
                    docker exec tools trivy fs /var/jenkins_home/workspace/Pipeline_VAmPI --format json --output /tools/trivy.json || {
                        echo "Erro ao executar Trivy.";
                        exit 1;
                    }

                    echo "Validando se o relatório Trivy foi gerado..."
                    if ! docker exec tools test -s /tools/trivy.json; then
                        echo "Erro: Relatório Trivy não foi gerado ou está vazio.";
                        exit 1;
                    fi

                    echo "Enviando relatório Trivy para DefectDojo..."
                    docker exec tools curl -X POST -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
                        -F "scan_type=Trivy Scan" \
                        -F "engagement=${ENGAGEMENT_ID}" \
                        -F "file=@/tools/trivy.json" \
                        -F "verified=false" \
                        -F "active=true" \
                        ${DEFECTDOJO_URL}/api/v2/import-scan/ || {
                        echo "Erro ao enviar relatório Trivy para DefectDojo.";
                        exit 1;
                    }

                    echo "Relatório Trivy Enviado com Sucesso!"
                    '''
                }
            }
        }
        stage('Análise de Segredos - Gitleaks') {
            steps {
                script {
                    sh '''
                    echo "Executando análise com Gitleaks..."
                    docker exec tools gitleaks detect --source=/var/jenkins_home/workspace/Pipeline_VAmPI --report-format=json --report-path=/tools/gitleaks.json || {
                        echo "Erro ao executar Gitleaks.";
                        exit 1;
                    }

                    echo "Validando se o relatório Gitleaks foi gerado..."
                    if ! docker exec tools test -s /tools/gitleaks.json; then
                        echo "Erro: Relatório Gitleaks não foi gerado ou está vazio.";
                        exit 1;
                    fi

                    echo "Enviando relatório Gitleaks para DefectDojo..."
                    docker exec tools curl -X POST -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
                        -F "scan_type=Gitleaks Scan" \
                        -F "engagement=${ENGAGEMENT_ID}" \
                        -F "file=@/tools/gitleaks.json" \
                        -F "verified=false" \
                        -F "active=true" \
                        ${DEFECTDOJO_URL}/api/v2/import-scan/ || {
                        echo "Erro ao enviar relatório Gitleaks para DefectDojo.";
                        exit 1;
                    }

                    echo "Relatório Gitleaks Enviado com Sucesso!"
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finalizado.'
        }
        failure {
            echo 'Pipeline falhou!'
        }
    }
}
