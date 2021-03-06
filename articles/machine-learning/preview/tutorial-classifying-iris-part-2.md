---
title: "Azure Machine Learning 서비스(미리 보기) 모델 작성 | Microsoft Docs"
description: "이 자습서 전체에서 Azure Machine Learning 서비스(미리 보기)를 사용하는 방법을 보여 줍니다. 2부이며 실험을 설명합니다."
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: tutorial
ms.date: 11/06/2017
ms.openlocfilehash: 3337ba9fa6845deb98813259821f77ff713f9eb1
ms.sourcegitcommit: 9890483687a2b28860ec179f5fd0a292cdf11d22
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/24/2018
---
# <a name="classify-iris-part-2-build-a-model"></a>아이리스 분류 2부: 모델 작성
Azure Machine Learning 서비스(미리 보기)는 데이터를 준비하고, 실험을 개발하고, 클라우드 범위에서 모델을 배포할 수 있는 전문 데이터 과학자를 위한 종단 간 데이터 과학 및 고급 분석 통합 솔루션입니다.

이 자습서는 3부로 구성된 시리즈 중 제2부입니다. 이 자습서의 부분에서는 Azure Machine Learning 서비스(미리 보기)를 사용하여 다음을 수행합니다.

> [!div class="checklist"]
> * Azure Machine Learning Workbench 사용
> * 스크립트 열기 및 코드 검토
> * 로컬 환경에서 스크립트 실행
> * 실행 기록 검토
> * 로컬 Docker 환경에서 스크립트 실행
> * 로컬 Azure CLI 창에서 스크립트 실행
> * 원격 Docker 환경에서 스크립트 실행
> * 클라우드 Azure HDInsight 환경에서 스크립트 실행

이 자습서에서는 변함 없는 [아이리스 꽃 데이터 집합](https://en.wikipedia.org/wiki/Iris_flower_data_set)을 사용합니다. 스크린샷은 Windows 전용이지만 Mac OS 환경에서도 거의 동일합니다.

## <a name="prerequisites"></a>필수 조건
먼저 이 자습서 시리즈의 제1부를 완료합니다. 이 자습서의 단계를 시작하기 전에 [데이터 준비 자습서](tutorial-classifying-iris-part-1.md)에 따라 Azure Machine Learning 리소스를 만들고 Azure Machine Learning Workbench 응용 프로그램을 설치합니다.

필요에 따라 로컬 Docker 컨테이너에 대해 스크립트를 실행하여 실험할 수 있습니다. 이렇게 하려면 Docker 엔진(Community Edition이면 충분)을 설치하고 Windows 또는 Mac OS 컴퓨터에서 로컬로 시작해야 합니다. Docker 설치에 대한 자세한 내용은 [Docker 설치 지침](https://docs.docker.com/engine/installation/)을 참조하세요.

원격 Azure VM 또는 Azure HDInsight Spark 클러스터의 Docker 컨테이너에서 실행할 스크립트를 디스패치하여 실험하기 위해 [Ubuntu 기반 Azure 데이터 과학 Virtual Machine 또는 HDInsight 클러스터를 만드는 지침](how-to-create-dsvm-hdi.md)을 수행할 수 있습니다.

## <a name="review-irissklearnpy-and-the-configuration-files"></a>iris_sklearn.py 및 구성 파일 검토
1. Azure Machine Learning Workbench 응용 프로그램을 열고, 자습서 시리즈의 이전 부분에서 만든 **myIris** 프로젝트를 엽니다.

2. 프로젝트가 열리면 가장 왼쪽 창에 있는 **파일** 단추(폴더 아이콘)를 선택하여 프로젝트 폴더의 파일 목록을 엽니다.

3. **iris_sklearn.py** 파일을 선택합니다. Python 코드가 Workbench의 새 텍스트 편집기 탭에서 열립니다.

   ![파일 열기](media/tutorial-classifying-iris/open_iris_sklearn.png)

   >[!NOTE]
   >이 샘플 프로젝트는 자주 업데이트되므로 사용자가 보는 코드가 이전 코드와 완전히 같지 않을 수도 있습니다.

4. 코딩 스타일에 익숙해지기 위해 Python 스크립트 코드를 검토합니다. 스크립트는 다음 작업을 수행합니다.

   - **iris.dprep** 데이터 준비 패키지를 로드하여 [pandas 데이터 프레임](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)을 만듭니다. 

        >[!NOTE]
        >여기서는 샘플 프로젝트와 함께 제공되는 `iris.dprep` 데이터 준비 패키지를 사용합니다. 이 패키지는 1부 자습서에서 작성한 `iris-1.dprep` 파일과 동일해야 합니다.

   - 임의의 기능을 추가하여 문제를 해결하기 어렵게 만듭니다. 아이리스는 거의 100% 정확도로 쉽게 분류할 수 있는 작은 데이터 집합이므로 임의성이 필요합니다.

   - [scikit-learn](http://scikit-learn.org/stable/index.html) 기계 학습 라이브러리를 사용하여 로지스틱 회귀 모델을 작성합니다. 

   - `outputs` 폴더의 파일에 [pickle](https://docs.python.org/2/library/pickle.html) 라이브러리를 삽입하여 모델을 직렬화합니다. 그런 다음 스크립트는 로드하고 메모리로 다시 역직렬화합니다.

   - 역직렬화된 모델을 사용하여 새 레코드에 대해 예측을 수행합니다. 

   - [matplotlib](https://matplotlib.org/) 라이브러리를 사용하여 두 그래프, 혼동 행렬 및 다중 클래스 ROC(receiver operating characteristic) 곡선을 그린 다음 `outputs` 폴더에 저장합니다.

   - `run_logger` 개체는 모델 정확도에 대한 정규화 속도를 로그로 기록하는 데 전체에 걸쳐 사용됩니다. 로그는 실행 기록에 자동으로 표시됩니다.


## <a name="execute-irissklearnpy-script-in-a-local-environment"></a>로컬 환경에서 iris_sklearn.py 스크립트 실행

처음으로 **iris_sklearn.py** 스크립트를 실행하도록 준비하겠습니다. 이 스크립트에는 **scikit-learn** 및 **matplotlib** 패키지가 필요합니다. **scikit-learn** 패키지는 Azure Machine Learning Workbench에 의해 이미 설치되었습니다. **matplotlib**는 여전히 설치해야 합니다. 

1. Azure Machine Learning Workbench에서 **파일** 메뉴를 선택한 다음 **명령 프롬프트 열기**를 선택하여 명령 프롬프트를 엽니다. 명령줄 인터페이스 창은 *Azure Machine Learning Workbench CLI 창* 또는 간단히 *CLI 창*이라고 합니다.

2. CLI 창에서 다음 명령을 입력하여 **matplotlib** Python 패키지를 설치합니다. 이 설치는 1분 이내 완료되어야 합니다.

   ```azurecli
   pip install matplotlib
   ```

   >[!NOTE]
   >이전 `pip install` 명령을 건너뛰는 경우 `iris_sklearn.py`의 코드 성공적으로 실행됩니다. `iris_sklearn.py`만을 실행하는 경우 코드는 기록 시각화에 표시된 대로 혼동 행렬 출력 및 다중 클래스 ROC 곡선 그래프를 생성하지 않습니다.

3. Workbench 앱 창으로 돌아갑니다. 

4. **iris_sklearn.py** 탭의 맨 위에 있는 도구 모음에서 **저장** 아이콘 옆의 드롭다운 메뉴를 선택하여 연 다음 **실행 구성**을 선택합니다. **로컬**을 실행 환경으로 선택한 다음 `iris_sklearn.py`를 실행할 스크립트로 입력합니다.

5. 다음으로 도구 모음의 오른쪽으로 이동하고 **인수** 필드에 `0.01`을 입력합니다. 

   ![제어 실행](media/tutorial-classifying-iris/run_control.png)

6. **실행** 단추를 선택합니다. 작업이 바로 예약됩니다. 이 작업은 Workbench 창의 오른쪽에 있는 **작업** 창에 나열됩니다. 

7. 몇 분 후에 작업 상태가 **제출 중**에서 **실행 중**으로, 마지막으로 **완료됨**으로 전환됩니다.

   ![sklearn 실행](media/tutorial-classifying-iris/run_sklearn.png)

8. **작업** 창의 작업 상태 텍스트에서 **완료됨**을 선택합니다. 팝업 창이 열리고 실행 중인 스크립트의 표준 출력(stdout) 텍스트가 표시됩니다. 표준 출력 텍스트를 닫으려면 팝업 의 오른쪽 위에 있는 **닫기**(**x**) 단추를 선택합니다.

9. **작업** 창의 동일한 작업 상태에서 **완료됨** 상태 및 시작 시간 바로 위에 있는 파란색의 **iris_sklearn.py [n]**(_n_은 실행 횟수) 텍스트를 선택합니다. **실행 속성** 창이 열리고 해당 특정 실행에 대한 다음 정보를 표시합니다.
   - **실행 속성** 정보
   - **출력** 파일
   - 있는 경우 **시각화**
   - **로그** 

   실행이 완료되면 팝업 창에서 다음과 같은 결과를 표시합니다.

   >[!NOTE]
   >이전에 학습 집합에 불규칙화를 도입했으므로 정확한 결과는 여기에 표시된 결과와 다를 수 있습니다.

   ```text
   Python version: 3.5.2 |Continuum Analytics, Inc.| (default, Jul  5 2016, 11:41:13) [MSC v.1900 64 bit (AMD64)]
   
   Iris dataset shape: (150, 5)
   Regularization rate is 0.01
   LogisticRegression(C=100.0, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
   Accuracy is 0.6792452830188679
   
   ==========================================
   Serialize and deserialize using the outputs folder.
   
   Export the model to model.pkl
   Import the model from model.pkl
   New sample: [[3.0, 3.6, 1.3, 0.25]]
   Predicted class is ['Iris-setosa']
   Plotting confusion matrix...
   Confusion matrix in text:
   [[50  0  0]
    [ 1 37 12]
    [ 0  4 46]]
   Confusion matrix plotted.
   Plotting ROC curve....
   ROC curve plotted.
   Confusion matrix and ROC curve plotted. See them in Run History details pane.
   ```

10. **실행 속성** 탭을 닫은 다음 **iris_sklearn.py** 탭으로 돌아갑니다. 

11. 추가 실행을 반복합니다. 

    **인수** 필드에 `0.001`에서 `10`까지 일련의 다른 숫자 값을 입력합니다. **실행**을 선택하여 코드를 몇 번 더 실행합니다. 인수 값이 변경될 때마다 해당 값이 코드의 로지스틱 회귀 알고리즘에 제공되므로 매번 다른 결과가 발생합니다.

## <a name="review-the-run-history-in-detail"></a>자세한 실행 기록 검토
Azure Machine Learning Workbench에서는 모든 스크립트 실행이 실행 기록 레코드로 캡처됩니다. **실행** 보기를 여는 경우 특정 스크립트의 실행 기록을 볼 수 있습니다.

1. **실행** 목록을 열려면 왼쪽 도구 모음에서 **실행** 단추(시계 아이콘)를 선택합니다. 그런 다음 **iris_sklearn.py**를 선택하여 `iris_sklearn.py`의 **실행 대시보드**를 표시합니다.

   ![실행 보기](media/tutorial-classifying-iris/run_view.png)

2. **실행 대시보드** 탭이 열립니다. 여러 실행에서 캡처된 통계를 검토합니다. 그래프는 탭의 위쪽에서 렌더링합니다. 각 실행에는 연속 번호가 있으며 실행 세부 정보는 화면 맨 아래의 테이블에 나열됩니다.

   ![실행 대시보드](media/tutorial-classifying-iris/run_dashboard.png)

3. 테이블을 필터링한 다음 그래프를 선택하여 각 실행의 상태, 기간, 정확도 및 정규화 속도를 봅니다. 

4. **실행** 테이블에서 2-3개의 실행을 선택하고 **비교** 단추를 선택하여 자세한 비교 창을 엽니다. 병렬 비교를 검토합니다. **비교** 창의 왼쪽 위에서 **실행 목록** 뒤로 단추를 선택하여 **실행 대시보드**로 돌아갑니다.

5. 개별 실행을 선택하여 실행 세부 정보 보기를 표시합니다. 선택한 실행에 대한 통계는 **실행 속성** 섹션에서 나열됩니다. 출력 폴더에 기록된 파일은 **출력** 섹션에서 나열되고 거기에서 파일을 다운로드할 수 있습니다.

   ![실행 세부 정보](media/tutorial-classifying-iris/run_details.png)

   두 개의 도면, 즉 혼돈 행렬 및 다중 클래스 ROC 곡선은 **시각화** 섹션에서 렌더링됩니다. 또한 모든 로그 파일은 **로그** 섹션에서 찾을 수 있습니다.

## <a name="execute-scripts-in-the-local-docker-environment"></a>로컬 Docker 환경에서 스크립트 실행

Machine Learning을 사용하면 Docker와 같은 추가 실행 환경을 쉽게 구성하고 이러한 환경에서 스크립트를 실행할 수 있습니다. 

>[!IMPORTANT]
>이 단계를 수행하려면 로컬에 설치되어 시작되는 Docker 엔진이 있어야 합니다. 자세한 내용은 Docker 설치 지침을 참조하세요.

1. 왼쪽 창에서 **폴더** 아이콘을 선택하여 프로젝트에 대한 **파일** 목록을 엽니다. `aml_config` 폴더를 펼칩니다. 

2. **docker-python**, **docker-spark** 및 **로컬**과 같이 미리 구성된 여러 환경이 있습니다. 

   각 환경에는 `docker-python.compute` 및 `docker-python.runconfig`와 같은 두 개의 파일이 있습니다. 텍스트 편집기에서 특정 옵션을 구성할 수 있는지 확인하려면 각 파일을 엽니다.  

   정리하려면 열려 있는 텍스트 편집기에 대한 탭에 있는 **닫기**(**x**)를 선택합니다.

3. **docker-python** 환경을 사용하여 **iris_sklearn.py** 스크립트를 실행합니다. 

   - 왼쪽 도구 모음에서 **시계** 아이콘을 선택하여 **실행** 창을 엽니다. **모든 실행**을 선택합니다. 
   - **모든 실행**  탭의 위쪽에서 대상이 되는 환경으로 기본 **로컬** 대신 **docker-python**을 선택합니다. 
   - 그런 다음 오른쪽으로 이동하고 실행할 스크립트로 **iris_sklearn.py**를 선택합니다. 
   - 스크립트에서 기본값을 지정하므로 **인수** 필드를 비워둡니다. 
   - **실행** 단추를 선택합니다.

4. 새 작업이 시작되는지 확인합니다. Workbench 창의 오른쪽에 있는 **작업** 창에 나타납니다.

   Docker를 처음 사용하는 경우 완료하는 데 몇 분 정도의 시간이 더 걸립니다. 

   내부적으로 Azure Machine Learning Workbench는 새 docker 파일을 빌드합니다. 
   새 파일은 `docker.compute` 파일에 지정된 기본 Docker 이미지 및 `conda_dependencies.yml` 파일에 지정된 종속성 Python 패키지를 참조합니다. 
   
   Docker 엔진은 다음 작업을 수행합니다.

    - Azure에서 기본 이미지를 다운로드합니다.
    - `conda_dependencies.yml` 파일에 지정된 Python 패키지를 설치합니다.
    - Docker 컨테이너를 시작합니다.
    - 실행 구성에 따라 프로젝트 폴더의 로컬 복사본을 복사 또는 참조합니다.      
    - `iris_sklearn.py` 스크립트를 실행합니다.

   마지막으로 **로컬**을 대상으로 지정할 때와 똑같은 결과가 표시됩니다.

5. 이제 Spark를 사용해 보겠습니다. Docker 기본 이미지에는 미리 설치되고 구성된 Spark 인스턴스가 포함되어 있습니다. 이 인스턴스로 인해 PySpark 스크립트를 실행할 수 있습니다. 이는 Spark를 직접 설치하고 구성하는 데 시간을 낭비하지 않고도 Spark 프로그램을 개발하고 테스트할 수 있는 쉬운 방법입니다. 

   `iris_spark.py` 파일을 엽니다. 이 스크립트에서는 `iris.csv` 데이터 파일을 로드하고 Spark Machine Learning 라이브러리의 로지스틱 회귀 알고리즘을 사용하여 아이리스 데이터 집합을 분류합니다. 이제 실행 환경을 **docker-spark**로 변경하고 스크립트를 **iris_spark.py**로 변경한 다음 다시 실행합니다. 이 프로세스는 Docker 컨테이너에서 Spark 세션을 만들고 시작해야 하기 때문에 시간이 약간 더 오래 걸립니다. 표준 출력(stdout)이 `iris_spark.py`의 표준 출력과 다름을 알 수도 있습니다.

6. 몇 가지 실행을 더 수행하고 서로 다른 인수를 사용하여 재생합니다. 

7. `iris_spark.py` 파일을 열어 Spark Machine Learning 라이브러리를 사용하여 작성된 로지스틱 회귀 모델을 봅니다. 

8. **작업** 패널, 실행 기록 목록 보기 및 다른 실행 환경의 실행 세부 정보 보기와 상호 작용합니다.

## <a name="execute-scripts-in-the-azure-machine-learning-cli-window"></a>Azure Machine Learning CLI 창에서 스크립트 실행

1. Azure Machine Learning Workbench에서 명령줄 창을 열고, **파일** 메뉴를 선택한 다음 **명령 프롬프트 열기**를 선택합니다. 명령 프롬프트가 프로젝트 폴더에서 `C:\Temp\myIris\>` 프롬프트와 함께 시작됩니다.

   >[!IMPORTANT]
   >다음 단계를 수행하려면 명령줄 창(Workbench에서 열림)을 사용해야 합니다.

2. 명령 프롬프트를 사용하여 Azure에 로그인합니다. 

   Workbench 응용 프로그램과 CLI는 Azure 리소스에 대해 인증할 때 독립적인 자격 증명 캐시를 사용합니다. 캐시된 토큰이 만료될 때까지 한 번만 수행하면 됩니다. **az account list** 명령은 로그인에 사용 가능한 구독 목록을 반환합니다. 둘 이상이 있는 경우 원하는 구독에서 ID 값을 사용합니다. 해당 구독을 기본 계정으로 설정하여 **az account set -s** 명령으로 사용한 다음 구독 ID 값을 제공합니다. 그런 다음 account **show** 명령을 사용하여 설정을 확인합니다.

   ```azurecli
   REM login by using the aka.ms/devicelogin site
   az login
   
   REM lists all Azure subscriptions you have access to 
   az account list -o table
   
   REM sets the current Azure subscription to the one you want to use
   az account set -s <subscriptionId>
   
   REM verifies that your current subscription is set correctly
   az account show
   ```

3. 인증을 마치고 현재 Azure 구독 컨텍스트가 설정되면 CLI 창에서 다음 명령을 입력하여 **matplotlib**을 설치한 다음 Python 스크립트를 실행할 실험으로 제출합니다.

   ```azurecli
   REM you don't need to run this command if you have installed matplotlib locally from the previous steps
   pip install matplotlib
   
   REM kicks off an execution of the iris_sklearn.py file against the local compute context
   az ml experiment submit -c local .\iris_sklearn.py
   ```

4. 출력을 검토합니다. Workbench를 사용하여 스크립트를 실행한 경우와 동일한 출력 및 결과를 갖습니다. 

5. Docker를 컴퓨터에 설치한 경우 Docker 실행 환경을 사용하여 동일한 스크립트를 다시 실행합니다.

   ```azurecli
   REM executes iris_sklearn.py in the local Docker container Python environment
   az ml experiment submit -c docker-python .\iris_sklearn.py 0.01
   
   REM executes iris_spark.py in the local Docker container Spark environment
   az ml experiment submit -c docker-spark .\iris_spark.py 0.1
   ```
6. Workbench에서 왼쪽 창의 **폴더** 아이콘을 선택하여 프로젝트 파일을 나열하고 **run.py**라는 Python 스크립트를 엽니다. 

   이 스크립트는 다양한 정규화 속도에서 반복하는 데 유용합니다. 이러한 속도로 실험을 여러 번 실행합니다. 이 스크립트는 `10.0`(터무니 없이 큰 숫자)의 정규화 속도로 `iris_sklearn.py` 작업을 시작합니다. 그런 다음 스크립트는 다음 실행에서 속도를 반으로 줄이고 속도가 `0.005`보다 작지 않을 때까지 계속합니다. 

   ```python
   # run.py
   import os
   
   reg = 10
   while reg > 0.005:
       os.system('az ml experiment submit -c local ./iris_sklearn.py {}'.format(reg))
       reg = reg / 2
   ```

   명령줄에서 **run.py** 스크립트를 열려면 다음 명령을 실행합니다.

   ```cmd
   REM submits iris_sklearn.py multiple times with different regularization rates
   python run.py
   ```

   `run.py`가 완료되면 Workbench의 실행 기록 목록 보기에서 그래프가 표시됩니다.

## <a name="execute-in-a-docker-container-on-a-remote-machine"></a>원격 컴퓨터의 Docker 컨테이너에서 실행
원격 Linux 컴퓨터의 Docker 컨테이너에서 스크립트를 실행하려면 해당 원격 컴퓨터에 대한 SSH 액세스(사용자 이름 및 암호) 권한이 있어야 합니다. 또한 원격 컴퓨터에는 Docker 엔진이 설치되어 실행되어야 합니다. 이러한 Linux 컴퓨터를 얻는 가장 쉬운 방법은 Azure에서 Ubuntu 기반 DSVM(데이터 과학 Virtual Machine)을 만드는 것입니다. [Azure ML Workbench에서 사용하는 Ubuntu DSVM를 만드는 방법](how-to-create-dsvm-hdi.md#create-an-ubuntu-dsvm-in-azure-portal)을 알아봅니다.

>[!NOTE] 
>CentOS 기반 DSVM은 지원되지 *않습니다*.

1. VM이 만들어지면 `.runconfig` 및 `.compute` 파일 쌍을 생성하는 경우 VM을 실행 환경으로 연결할 수 있습니다. 다음 명령을 사용하여 파일을 생성합니다. 새 환경 이름을 `myvm`으로 지정하겠습니다.
 
   ```azurecli
   REM creates an myvm compute target
   az ml computetarget attach remotedocker --name myvm --address <IP address> --username <username> --password <password>
   ```
   
   >[!NOTE]
   >IP 주소는 `vm-name.southcentralus.cloudapp.azure.com`과 같이 공개적으로 주소 지정이 가능한 FQDN(정규화된 도메인 이름)일 수도 있습니다. DSVM에 FQDN을 추가하고 IP 주소 대신 여기에서 사용하는 것이 좋습니다. 비용을 절약하기 위해 특정 시점에 VM을 끌 수 있으므로 이 방법은 좋습니다. 또한 다음에 VM을 시작할 때 IP 주소가 변경되었을 수도도 있습니다.

   다음으로 VM에 Docker 이미지를 생성하여 스크립트 실행을 준비하는 다음 명령을 실행합니다.
   
   ```azurecli
   REM prepares the myvm compute target
   az ml experiment prepare -c myvm
   ```
   >[!NOTE]
   >`myvm.runconfig`의 `PrepareEnvironment` 값을 기본 `false`에서 `true`로 변경할 수도 있습니다. 이 변경은 첫 번째 실행에서 Docker 컨테이너를 자동으로 준비합니다.

2. `aml_config` 아래에 생성된 `myvm.runconfig` 파일을 편집하고 프레임워크를 기본 `PySpark`에서 `Python`으로 변경합니다.

   ```yaml
   "Framework": "Python"
   ```
   >[!NOTE]
   >프레임워크 설정을 PySpark로 둔 경우 역시 작동합니다. 그러나 Python 스크립트를 실행하는 데 Spark 세션이 실제로 필요하지 않으면 약간 비효율적입니다.

3. _myvm_을 대상으로 지정한 경우를 제외하고는 CLI 창에서 이전에 수행한 것과 동일한 명령을 실행합니다.
   ```azurecli
   REM executes iris_sklearn.py in a remote Docker container
   az ml experiment submit -c myvm iris_sklearn.py
   ```
   이 명령은 원격 Linux VM에서 실행된다는 것을 제외하고는 `docker-python` 환경에 있는 것처럼 실행됩니다. CLI 창에서 동일한 출력 정보가 표시됩니다.

4. 컨테이너에서 Spark를 사용해 보겠습니다. 파일 탐색기를 엽니다. 기본 파일 조작 명령에 익숙한 경우 CLI 창에서도 이 작업을 수행할 수 있습니다. `myvm.runconfig` 파일의 복사본을 만들고 이름을 `myvm-spark.runconfig`로 지정합니다. 새 파일을 편집하여 `Framework` 설정을 `Python`에서 `PySpark`로 변경합니다.
   ```yaml
   "Framework": "PySpark"
   ```
   `myvm.compute` 파일은 변경하지 마세요. 동일한 VM의 동일한 Docker 이미지가 Spark 실행에 사용됩니다. 새 `myvm-spark.runconfig`에서 `target` 필드는 `myvm`이라는 이름을 통해 동일한 `myvm.compute` 파일을 가리킵니다.

5. 다음 명령을 입력하여 원격 Docker 컨테이너의 Spark 인스턴스에서 실행합니다.
   ```azureli
   REM executes iris_spark.py in a Spark instance on a remote Docker container
   az ml experiment submit -c myvm-spark .\iris_spark.py
   ```

## <a name="execute-script-in-an-hdinsight-cluster"></a>HDInsight 클러스터에서 스크립트 실행
이 스크립트는 HDInsight Spark 클러스터에서도 실행할 수 있습니다. [Azure ML Workbench에서 사용하는 HDInsight Spark Cluster를 만드는 방법](how-to-create-dsvm-hdi.md#create-an-apache-spark-for-azure-hdinsight-cluster-in-azure-portal)을 알아봅니다.

>[!NOTE] 
>HDInsight 클러스터는 Azure Blob을 주 저장소로 사용해야 합니다. 아직 Azure Data Lake Storage를 사용하도록 지원되지 않습니다.

1. Azure HDInsight용 Spark 클러스터에 액세스할 수 있으면 다음과 같이 HDInsight 실행 구성 명령을 생성합니다. HDInsight 클러스터 이름, HDInsight 사용자 이름 및 암호를 매개 변수로 제공합니다. 다음 명령을 사용합니다.

   ```azurecli
   REM creates a compute target that points to a HDInsight cluster
   az ml computetarget attach cluster --name myhdi --address <cluster head node FQDN> --username <username> --password <password>

   REM prepares the HDInsight cluster
   az ml experiment prepare -c myhdi
   ```

   클러스터 헤드 노드 FQDN은 일반적으로 `<cluster_name>-ssh.azurehdinsight.net`입니다.

   >[!NOTE]
   >`username`은 클러스터 SSH 사용자 이름입니다. HDInsight 설치 중에 변경하지 않으면 기본값은 `sshuser`입니다. 값은 설치 중에 만들어져 클러스터의 관리 웹 사이트에 액세스할 수 있도록 설정되는 다른 사용자인 `admin`이 아닙니다. 

2. 다음 명령을 실행하여 HDInsight 클러스터에서 스크립트를 실행합니다.

   ```azurecli
   REM executes iris_spark on the HDInsight cluster
   az ml experiment submit -c myhdi .\iris_spark.py
   ```

   >[!NOTE]
   >원격 HDInsight 클러스터에 대해 실행하는 경우 `admin` 사용자 계정을 사용하여 `https://<cluster_name>.azurehdinsight.net/yarnui`에서 YARN(Yet Another Resource Negotiator) 작업 실행 세부 정보를 볼 수도 있습니다.


## <a name="next-steps"></a>다음 단계
3부 자습서 시리즈의 제2부에서는 Azure Machine Learning 서비스를 사용하여 다음을 수행하는 방법을 알아보았습니다.
> [!div class="checklist"]
> * Azure Machine Learning Workbench 사용
> * 스크립트 열기 및 코드 검토
> * 로컬 환경에서 스크립트 실행
> * 실행 기록 검토
> * 로컬 Docker 환경에서 스크립트 실행
> * 로컬 Azure CLI 창에서 스크립트 실행
> * 원격 Docker 환경에서 스크립트 실행
> * 클라우드 HDInsight 환경에서 스크립트 실행

시리즈의 제3부로 이동할 준비가 되었습니다. 이제 로지스틱 회귀 모델을 만들었으므로 실시간 웹 서비스로 배포해 보겠습니다.

> [!div class="nextstepaction"]
> [모델 배포](tutorial-classifying-iris-part-3.md)
