# converting a clojure project to aws lambda runtime

## Background

<img src='../lib/accountant.png' width=400>

My life, 14 hours a day and 6 days a week. Why am I alive?

<img src='../lib/raspberry_pi.png' width=400>

Best $50 investment I've made in my career

<img src='../lib/funding_circle.png' width=400>

Funding Circle hired me as an engineer in June 2016. Get in contact with us at
<https://www.fundingcircle.com/us/about/careers/>, <techrecruit@fundingcircle.com>
or <john.skilbeck@fundingcircle.com>

<img src='../lib/clojure.png' width=400>

I was told during the interview we'd be programming in Clojure. What is Clojure?
I think I've heard of closures before. Something with scope? It's a LISP? Isn't
that something ultra-neckbeards philosophize about but can't actually build
anything with?

Fast forward two years...

## markets-etl

Wanted to set up an ETL of financial data, not just equity prices, but also bond
yields, risk free rate of return, GDP data, currency prices, and real estate
indecies by zip code.

Quandl has this data available in an API, so wrote a Clojure wrapper around that
and began importing that data onto a small postgres server on a raspberry pi on
my own network in my apartment.

However, in my apartment, I don't have the fancy distributed systems technologies
(Apache Mesos) we have at work.

### source code, project layout

```bash
@mbp:markets-etl $ tree -I 'test|dev-resources|deploy'
.
├── LICENSE
├── README.md
├── project.clj
└── src
    ├── jobs
    │   ├── currency.clj
    │   ├── economics.clj
    │   ├── equities.clj
    │   ├── interest_rates.clj
    │   └── real_estate.clj
    └── markets_etl
        ├── api.clj
        ├── sql.clj
        └── util.clj
```

in `src/`, `jobs/` which are the business logic, and `markets_etl/` which act
as internal libraries for the project, such as an api wrapper, a sql
wrapper, and shared utility functions

A sample job:

```clojure
(ns jobs.currency
  (:require [clojure.java.jdbc :as jdbc]
            [markets-etl.api :as api]
            [markets-etl.sql :as sql]
            [markets-etl.util :as util])
  (:gen-class))

(def datasets
  '({:dataset "CURRFX"
     :ticker ["EURUSD" "GBPUSD"]}))

(def query-params
  {:limit      20
   :start_date util/last-week
   :end_date   util/now})

(defn execute! [cxn data]
  (jdbc/with-db-transaction [txn cxn]
    (->> data
         (map prepare-row)
         flatten
         (map #(update-or-insert! txn %))
         doall)))

(defn -main [& args]
  (jdbc/with-db-connection [cxn (-> :jdbc-db-uri env)]
    (let [get-data (fn [{:keys [dataset
                                ticker]}]
                     (->> ticker
                          (map (fn [tkr]
                                 (-> (api/query-quandl! dataset
                                                        tkr
                                                        query-params)
                                     (assoc :dataset dataset :ticker tkr))))))
          data        (->> datasets
                           (map get-data)
                           flatten)]

      (execute! cxn data)))
```

The result:

```
@C02NN3NBG3QT:~ $ psql $db_uri -c 'select * from dw.currency limit 5'

 dataset | ticker | currency |    date    |       rate       |     high_est     |     low_est
---------+--------+----------+------------+------------------+------------------+------------------
 CURRFX  | EURUSD | EUR      | 2017-09-24 | 1.19271969795230 | 1.19388735294340 | 1.18518519401550
 CURRFX  | EURUSD | EUR      | 2017-10-03 | 1.17475688457490 | 1.17867541313170 | 1.17474305629730
 CURRFX  | EURUSD | EUR      | 2017-09-17 | 1.19371628761290 | 1.19688808918000 | 1.19168210029600
 CURRFX  | GBPUSD | GBP      | 2017-09-24 | 1.35073077678680 | 1.35701787471770 | 1.34634804725650
 CURRFX  | GBPUSD | GBP      | 2017-09-17 | 1.35751521587370 | 1.36156308650970 | 1.34825396537780
(5 rows)

```

### deployment strategy

```bash
@mbp:markets-etl $ tree deploy/
deploy/
├── bin
│   ├── run-docker
│   └── run-job
├── default
│   ├── Dockerfile
│   ├── Dockerfile.arm
│   ├── post-receive
│   └── publish-image
├── qemu
│   └── qemu-arm-static
└── tasks
    └── crontab
```

CI & deploy strategy is with CircleCI / quay.io / Docker / bare metal on both
ARM and x86 platforms. For any PR, Circle will check out the repo and use the
`publish-image` script to create an x86 and ARM docker container and publish
those to quay.io.

`circle.yml`:
```
machine:
  services:
    - docker
    - postgresql

test:
  override:
    - case $CIRCLE_NODE_INDEX in 0) lein cljfmt check ;; 1) lein test ;; esac:
        parallel: true

deployment:
  quay:
    branch: /.*/
    commands:
      - deploy/bin/publish-image
```

`deploy/bin/publish-image`:
```bash
#!/usr/bin/env bash
set -eou pipefail

publish_x86() {
  if [[ -f Dockerfile ]]; then rm Dockerfile; fi
  if [[ -d target     ]]; then rm -rf target; fi

  cat deploy/default/publish-image | \
    sed "s/tag='latest'/tag='x86'/g" | \
    sed 's/$image:$CIRCLE_SHA1/$image:$CIRCLE_SHA1-x86/g' | \
    sed 's/tag="${CIRCLE_BRANCH}_${CIRCLE_BUILD_NUM}"/tag="${CIRCLE_BRANCH}_${CIRCLE_BUILD_NUM}_x86"/g' \
    >>deploy/bin/publish-x86

  chmod u+x deploy/bin/publish-x86 && \
    deploy/bin/publish-x86
}

publish_arm() {
  if [[ -f Dockerfile ]]; then rm Dockerfile; fi
  if [[ -d target     ]]; then rm -rf target; fi

  cat deploy/default/publish-image | \
    sed "s/cat '.\/deploy\/default\/Dockerfile'/cat '.\/deploy\/default\/Dockerfile.arm'/g" | \
    sed "s/tag='latest'/tag='arm'/g" | \
    sed 's/$image:$CIRCLE_SHA1/$image:$CIRCLE_SHA1-arm/g' | \
    sed 's/tag="${CIRCLE_BRANCH}_${CIRCLE_BUILD_NUM}"/tag="${CIRCLE_BRANCH}_${CIRCLE_BUILD_NUM}_arm"/g' \
    >>deploy/bin/publish-arm

  chmod u+x deploy/bin/publish-arm && \
    deploy/bin/publish-arm
}

publish_arm && \
  publish_x86
```

`deploy/default/publish-image`:
```bash
#!/usr/bin/env bash
set -euo pipefail

app=$(basename $(pwd))
registry='quay.io'
image="${registry}/skilbjo/${app}"
email='skilbjo@github.com'

cat './deploy/default/Dockerfile' >Dockerfile

if [[ $CIRCLE_BRANCH == master ]]; then
  tag='latest'
else
  tag="${CIRCLE_BRANCH}_${CIRCLE_BUILD_NUM}"
fi

echo $(git rev-parse HEAD) >.tag

docker login -u $QUAY_ROBOT_USERNAME -p $QUAY_ROBOT_PASSWORD -e $email $registry
lein uberjar
docker build --rm -t $image:$tag .
docker tag $image:$tag $image:$CIRCLE_SHA1
docker push $image
```
`deploy/default/post-receive`:
```bash
#!/usr/bin/env bash
set -eou pipefail

user=$(whoami)
dir="/home/${user}/deploy/app"
app=$(basename $(pwd) | sed -e 's/.git//')
deploy_dir="$dir/$app"

GIT_WORK_TREE="$deploy_dir" git checkout -f

mkdir -p "$deploy_dir" && cd "$deploy_dir"

## build steps here ##
case $(uname) in
  (Linux)
    sudo cp deploy/tasks/crontab "/etc/cron.d/$app" ;;
  (FreeBSD)
    sudo cp deploy/tasks/crontab "/var/cron/tabs/$app" ;;
esac
```

A `post-receive` git hook lives on target machines which checks out the code
and places the `crontab` file in necessary directories (FreeBSD in `/var/cron/tabs/`
and Linux in `/etc/cron.d/`. Note: if you place in these directores, cron will
poll the files every minute to see if it needs to execute; no need to edit user-
specific crontab files. Cron file has runtime information and also cleans
the docker cache (`docker images | xargs docker rmi`) for automated pulls from
quay.io for new commits to master.

The cron file:

```bash
## -- variables -----------------------------------
  SHELL=/bin/bash
  PATH=:/bin:/usr/bin:/usr/local/bin:/usr/sbin:/usr/local/sbin
  MAILTO=john.skilbeck@gmail.com
  cmd="deploy/bin/run-docker"
  app_dir="/home/skilbjo/deploy/app/markets-etl"

## -- jobs ----------------------------------------
0 7 * * * skilbjo . /home/skilbjo/.profilelocal; cd "$app_dir" ; $cmd -m jobs.equities >/dev/null 2>&1
0 6 * * * skilbjo . /home/skilbjo/.profilelocal; cd "$app_dir" ; $cmd -m jobs.currency >/dev/null 2>&1
0 6 */4 * * skilbjo . /home/skilbjo/.profilelocal; cd "$app_dir" ; $cmd -m jobs.economics >/dev/null 2>&1
0 6 */4 * * skilbjo . /home/skilbjo/.profilelocal; cd "$app_dir" ; $cmd -m jobs.interest-rates >/dev/null 2>&1
0 6 */15 * * skilbjo . /home/skilbjo/.profilelocal; cd "$app_dir" ; $cmd -m jobs.real-estate >/dev/null 2>&1

## -- clear the cache -----------------------------
0 3 */20 */2 * skilbjo function _(){ docker images | awk '{print $3}' | grep -v IMAGE | xargs docker rmi --force;};_ >/dev/null
```

The entrypoint is the `bin/run-docker` script

```bash
#!/usr/bin/env bash
set -eou pipefail

case $(uname -a) in
  *amd64* | *x_86* | *x86_64* )
    arch="x86" ;;
  *arm* )
    arch="arm" ;;
esac

img="quay.io/skilbjo/$app_name:$arch"
job_cmd="usr/local/deploy/bin/run-job"

docker run --rm \
  -e jdbc_db_uri="$(echo $jdbc_db_uri)" \
  -e quandl_api_key="$(echo $quandl_api_key)" \
  -e healthchecks_io_api_key="$(echo $healthchecks_io_api_key)" \
  "$img" \
  "$job_cmd" $@
```

which calls the `bin/run-job` script once in the container to launch the jar file.

```bash
#!/usr/bin/env bash
set -eou pipefail

java_opts="-Xms256m \
           -Xmx512m \
           -XX:MaxMetaspaceSize=128m \
           -server \
           -Duser.timezone=PST8PDT"
cmd="java $java_opts -jar app.jar"

# Prereqs
set +e
apk fix || echo 'Unable to reach apk... continuing...'
set -e

exec $cmd $@
```

## aws lambda

EC2 containers, virtualized computers. But what about when you only need it now
and then? Script to go launch EC2 and run your job, and then shut down? You're
paying for all that time.

AWS realizes a split in what computers do: compute, internet, and storage. Cloud,
revolutionize, save the world, big data, etc etc.

So, to migrate from apartment to cloud, yet still free (AWS Lamba is 1M free
requests per month), I started to investigate.

AWS Lambda quick facts:

- 2014 product launches
- initially only node.js, max 1 minute runtime, no VPC support
- mid-2015 java8, max 5 minute runtime
- launches LXC container (LXC is similar to docker containers) when invoked
- billed in 100ms increments, pay for startup
- openjdk 1.8
- no control of startup flags, last two flags make startups less painful

```bash
java -XX:MaxHeapSize=85% of configured Lambda memory \
     -XX:UseSerialGC \         # overriding default garbage collector with Serial GC
     -XX:+TieredCompilation \
     -Xshare:on \              # Class data sharing
     -jar app.jar
```

### turn a project meant for a metal runtime to an aws runtime

No more control of the main it calls. From this:

```bash
java -Xms256m \
     -Xmx512m \
     -XX:MaxMetaspaceSize=128m \
     -server \
     -Duser.timezone=PST8PDT \
     -jar app.jar \
     -m jobs.currency
```

```bash
java -XX:MaxHeapSize=85% of configured Lambda memory \
     -XX:UseSerialGC \         # overriding default garbage collector with Serial GC
     -XX:+TieredCompilation \
     -Xshare:on \              # Class data sharing
     -jar app.jar
```

So, entrypoint is all off. What is the entrypoint? Unlike control of the cron
script, which calls `java -jar /app.jar -m jobs.currency`, there is no control.
Give up?

Actually, you tell AWS Lambda what the entrypoint is. Note this flag in the aws
cli call from above: `--handler "jobs.aws-lambda"`

<img src='../lib/aws_lambda.png' width=800>

Then create a clojure entrypoint that wraps the orchestration of the other
entrypoints.

```clojure
(ns jobs.aws-lambda
  (:require [jobs.currency :as currency]
            [jobs.economics :as economics]
            [jobs.equities :as equities]
            [jobs.interest-rates :as interest-rates]
            [jobs.real-estate :as real-estate]
            [markets-etl.util :as util])
  (:gen-class
    :name "jobs.aws-lambda"
    :implements [com.amazonaws.services.lambda.runtime.RequestStreamHandler]))

(defn main []
  (let [_              (println "Starting jobs... ")

        currency       (currency/-main)
        economics      (economics/-main)
        equities       (equities/-main)
        interest-rates (interest-rates/-main)
        real-estate    (real-estate/-main)

        _              (println "Finished!")]))

(defn -handleRequest [_ event _ context]
  (let [event' (-> event
                   io/reader
                   (json/read :key-fn keyword))]
    (main)))
```

<img src='../lib/cloudwatch_logs.png' width=800>

Script for doing various changes to the AWS environment:
- include orchestration main (`aws_lambda.clj`)
- include a few AWS java libraries (to take in a lambda stream)
- change environment variables to use AWS KMS (so no secrets in plaintext)

```bash
#!/usr/bin/env bash
set -eou pipefail

copy_files(){
  cleanup

  cp "${src_dir}/project.clj"  "${app}/project.clj"
  cp -r "${src_dir}/src"       "${app}/src"
}

add_aws_lambda_clj(){
  cp "${app}/include/aws_lambda.clj" "${app}/src/jobs/aws_lambda.clj"
}

util_clj(){
  sed -i '' -e '1,7d;' "${app}/src/markets_etl/util.clj"

  local header='(ns markets-etl.util
  (:require [clj-http.client :as http]
            [clj-time.format :as formatter]
            [clojure.string :as string])
  (:import [com.amazonaws.services.kms AWSKMS AWSKMSClientBuilder]
           [com.amazonaws.services.kms.model DecryptRequest]
           [java.util Base64]
           (java.nio ByteBuffer)
           (java.nio.charset Charset)))'

  echo "$header" | cat - "${app}/src/markets_etl/util.clj" >temp && \
    mv temp "${app}/src/markets_etl/util.clj"

  local decrypt='
; -- aws -----------------------------------------------
(defn decrypt [ciphertext]
  (let [decoder (Base64/getDecoder)
        decoded-text (.decode decoder ciphertext)
        kms-client (AWSKMSClientBuilder/defaultClient)
        decode-request (doto (DecryptRequest.)
                         (.withCiphertextBlob (ByteBuffer/wrap decoded-text)))
        decode-response (.decrypt kms-client decode-request)]
    (.toString (.decode (Charset/forName "UTF-8") (.getPlaintext decode-response)))))'

  echo "$decrypt" >>"${app}/src/markets_etl/util.clj"
}

project_clj(){
  sed -i '' -e '1,3d;' "${app}/project.clj"

  local header='(defproject markets-etl "0.1.0"
  :uberjar-name "markets-etl.jar"
  :dependencies [[org.clojure/clojure "1.8.0"]
                 [com.amazonaws/aws-lambda-java-core "1.0.0"]
                 [com.amazonaws/aws-java-sdk-kms "1.11.98"]
                 [amazonica "0.3.117" :exclusions [com.amazonaws/aws-java-sdk]]'

  echo "$header" | cat - "${app}/project.clj" >temp && \
    mv temp "${app}/project.clj"
}

src_files_clj(){
  # wrap decrypt
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/jobs/currency.clj"
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/jobs/economics.clj"
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/jobs/equities.clj"
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/jobs/interest_rates.clj"
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/jobs/real_estate.clj"

  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/markets_etl/api.clj"
  sed -i '' 's/ env/ env util\/decrypt/g' "${app}/src/markets_etl/sql.clj"
}

add_lambda_wrappers(){
  util_clj
  project_clj
  src_files_clj
}

copy_files && \
  add_aws_lambda_clj
  add_lambda_wrappers
```

And a script for building the project and uploading it to AWS:

```bash
#!/usr/bin/env bash
set -eou pipefail

build() {
  deploy/build-project

  lein uberjar && \
    cp "target/uberjar/${app}.jar" app.jar
}

new(){
  build

  aws --profile personal \
    lambda create-function \
    --region us-east-1 \
    --function-name "${app}" \
    --zip-file 'fileb://app.jar' \
    --role arn:aws:iam::493240119367:role/lambda_with_athena \
    --handler "jobs.aws-lambda" \
    --runtime java8 \
    --profile default \
    --timeout 10 \
    --memory-size 360
}

new
```

<img src='../lib/lambda_env_vars.png' width=800>

For scheduling, set a cloudwatch rule to call the entrypoint once a day.
<img src='../lib/cloudwatch.png' width=800>

## Additional Follow Ups (another talk?)
- RDS only free for 1 year. However, as AWS Lambda is to EC2, AWS Athena is to
RDS databases, by using S3 and Presto (similar to Hadoop / Hive)- to create a SQL
interface on text files.
- AWS Athena pricing is $5 per terabyte scanned, with a minimum of 10mb. Equivalent
pricing for small queries is $0.00005 per query, meaning price is $0.01 per
2,000 queries. Equivalent RDS pricing (smallest instance: t1.micro: $150 / year).
- AWS Lambda to update metastore partitions when new file added
- web server to use AWS Athena JDBC driver to get data

```bash
@mbp:~ $ curl -s https://skilbjo-aws.duckdns.org/api/currency/latest | \
  jq '.body[] | select(.currency == "GBP")'
{
  "date": "2018-01-17T00:00:00Z",
  "rate": "1.3798813819885",
  "high_est": "1.3844662904739",
  "low_est": "1.3758565187454",
  "dataset": "CURRFX",
  "ticker": "GBPUSD",
  "currency": "GBP",
  "s3uploaddate": "2018-01-20T08:00:00Z"
}
```

### Conclusion / Questions?

```
 _______________________________________
/ Thanks🍺 for learning about Clojure \
| and AWS Lambda with me!               |
\ 🎉🎉🎉🎉🎉🎉🎉🎉🎉                  /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
@C02NN3NBG3QT:~ $
