# 对接用户绑定注册信息同步脚本

[TOC]

## 项目描述
- 项目名称：百迈克定制
- 启动时间：2017.7.24
- ES 版本：x8 <code>8.0.22</code>
- 流程图：

1. 先检查百迈克的用户是否已经绑定到es中，绑定过就更新用户的信息，未绑定就注册用户并绑定用户，然后同步用户信息．
2. 需要同步的信息(用户真实姓名，昵称，手机号，邮箱，头像，积分)
3.　同步规则: 要求显示真实姓名（即昵称为真实姓名，重复则增加三个不重复的随机字符串），头像显示url的图像，积分同步到es的虚拟币账户（cash_account表）
4. 导入数据文件app/data/baimaike_users.csv，脚本：src/CustomBundle/Command/BatchImportCommand.php
5. 对于数百万条数据量的CSV文件，文件大小可能达到数百M，如果简单读取的话很可能出现超时或者卡死的现象，可以分批次打开文件(执行脚本传入没批次执行的行数line)．

## 同步文件

```
id,username,real_name,email,phone,photo_url,beans_count
10,zuming,刘xx,liuxax@biomarker.com.cn,18610808911,http://img.biocloud.cn/upload_image/usericon_publish/1500610643246.png,200
12,liug,刘xx,liugas@biomarker.com.cn,13211111111,http://img.biocloud.cn/upload_image/usericon_publish/1470295292479.png,566
......

```

## 同步脚本

```

<?php

namespace CustomBundle\Command;

use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Filesystem\Filesystem;
use AppBundle\Command\BaseCommand;
use AppBundle\Common\SimpleValidator;
use ApiBundle\Api\Resource\Auth\AuthAccessToken;

class BatchImportCommand extends BaseCommand
{
    protected function configure()
    {
        $this->addArgument(
            'line',
            InputArgument::OPTIONAL,
            '分批量导入行数（默认全部行）',
            0
        )
        ->setName('baimaike:import')
        ->setDescription('百迈客用户批量导入绑定(数据默认放在app/data/baimaike_users.csv)');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $output->writeln('<info>开始读取文件...</info>');
        $line = $input->getArgument('line', 0);

        $rootDir = $this->getBiz()->offsetGet('kernel.root_dir');
        $filepath = $rootDir.'/data/baimaike_users.csv';
        $errorFilepath = $rootDir.'/data/baimaike_users_error.csv';
        $filesystem = new Filesystem();
        if (!$filesystem->exists($filepath)) {
            $output->writeln('<info>文件不存在</info>');
        }
        if ($filesystem->exists($errorFilepath)) {
            $filesystem->remove($errorFilepath);
        }

        $contents = file($filepath);
        $totalLines = count($contents) - 1;
        $count = $line ? $line : $totalLines;
        $errorCount = 0;
        $successCount = 0;
        $starTime = time();

        $output->writeln("<info>开始导入用信息，总用户{$totalLines}人</info>");
        $lines = ceil($totalLines / $count);

        for ($i = 1; $i <= $lines; $i++) {
            $res = $this->csvGetLines($filepath, $count, ($i - 1) * $count);
            $resCount = $this->bindUsers($res);
            $errorCount   += $resCount[0];
            $successCount += $resCount[1];
        }
        $time = time() - $starTime;
        $output->writeln("<info>导入用户结束，总耗时：{$time}，成功导入人数{$successCount}，失败人数{$errorCount}人</info>");
    }

    protected function bindUsers($users)
    {
        $errorCount = 0;
        $successCount = 0;

        foreach ($users as $userData) {
            if ($userData) {
                $fields['user_id'] = $userData[0];
                $fields['nickname'] = $userData[1];
                $fields['truename'] = $userData[2];
                $fields['email'] = $userData[3];
                $fields['mobile'] = $userData[4];
                $fields['avater'] = $userData[5];
                $fields['coin'] = $userData[6];

                $user = $this->bindUser($fields);
                if (!$user) {
                    $errorCount++;
                } else {
                    $successCount++;
                }
            }
        }

        return array($errorCount, $successCount);
    }

    private function bindUser($fields)
    {
        $userBind = $this->getUserService()->getUserBindByTypeAndFromId(AuthAccessToken::BIND_TYPE, $fields['user_id']);

        if (!empty($userBind)) {
            // echo "已经绑定过: {$fields['user_id']}";
            // echo "\n";
            return $this->getUserService()->getUser($userBind['toId']);
        }

        $registration = $this->getRegisterData($fields);
        $biz = $this->getBiz();
        try {
            $biz['db']->beginTransaction();

            $user = $this->getAuthService()->register($registration);
            $this->getUserService()->bindUser(AuthAccessToken::BIND_TYPE, $fields['user_id'], $user['id'], '');
            $this->updateUserInfo($user['id'], $fields);

            $biz['db']->commit();
        } catch (\Exception $e) {
            $biz['db']->rollback();
            echo "----------------------------------------------";
            echo "\n";
            echo '错误用户id: '.$fields['user_id'];
            echo "\n";
            var_dump($registration);
            echo "\n";
            echo '错误信息：'.$e->getMessage();
            echo "\n";
            $this->exportErrorCvs($fields);
            return false;
        }
        
        return $user;
    }

    private function updateUserInfo($userId, $fields)
    {
        if (!empty($fields['avater'])) {
            $this->getUserService()->changeAvatar($userId, $fields['avater']);
        }

        if (!empty($fields['truename'])) {
            $this->getUserService()->updateUserProfile($userId, array(
                'truename' => $fields['truename'],
            ));
        }

        if (!empty($fields['mobile'])) {
            $this->getUserService()->updateUserProfile($userId, array(
                'mobile'   => $fields['mobile']
            ));
        }
    }
 
    private function getRegisterData($formData)
    {
        if ($this->getUserService()->isNicknameAvaliable($formData['truename'])) {
            $userData['nickname']  = $formData['truename'];
        } else {
            $userData['nickname']  = $formData['truename'].substr(md5(microtime(true)), 0, 3);
        }

        if (empty($formData['email'])) {
            $formData['email'] = $this->getUserService()->generateEmail($formData);
        }
        
        $userData['password']  = $this->getRandomChar();
        $userData['createdIp'] = '';
        $userData['type']      = 'api_import';
        $userData['email']     = $formData['email'];
        if (SimpleValidator::mobile($formData['mobile'])) {
            $userData['verifiedMobile'] = $formData['mobile'];
        }

        return $userData;
    }

    private function getRandomChar()
    {
        return base_convert(sha1(uniqid(mt_rand(), true)), 16, 36);
    }

        /**
     * csvGetLines 读取CSV文件中的某几行数据
     * @param $csvfile csv文件路径
     * @param $lines 读取行数
     * @param $offset 起始行数
     * @return array
     * */
    protected function csvGetLines($csvfile, $lines, $offset = 0)
    {
        if (!$fp = fopen($csvfile, 'r')) {
            return false;
        }
        $i = $j = 0;
        while (false !== ($line = fgets($fp))) {
            if ($i++ < $offset) {
                continue;
            }
            break;
        }
        $data = array();
        while (($j++ < $lines) && !feof($fp)) {
            $data[] = fgetcsv($fp, 1000, ',');
        }
        fclose($fp);
        return $data;
    }

    protected function exportErrorCvs($data)
    {
        $rootDir = $this->getBiz()->offsetGet('kernel.root_dir');
        $filepath = $rootDir.'/data/baimaike_users_error.csv';
        $file = fopen($filepath, 'a+b');
        fputcsv($file, $data);
        fclose($file);
    }

    private function getAuthService()
    {
        return $this->createService('User:AuthService');
    }

    private function getCashAccountService()
    {
        return $this->createService('Cash:CashAccountService');
    }
}
```