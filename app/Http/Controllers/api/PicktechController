<?php

namespace App\Http\Controllers\api;

use App\Http\Controllers\Controller;
use App\Models\Attendances;
use App\Models\Block_steps;
use App\Models\Npick_block_withdraws;
use App\Models\Npick_blocks;
use App\Models\Settings;
use App\Models\Users;
use GuzzleHttp\Client;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;
use Mail;

class PicktechController extends Controller
{
    /**
     * N+ 포인트 출석 내역 조회
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function history_view(Request $request)
    {
        $user = Auth()->user();

        $user_id =  $request->input('user_id') ? $request->input('user_id') : $user->id;

        $attendanc_cnt = Attendances::select('id')->where([['user_id',$user_id]])->count();
        $today_ck = Attendances::where([['user_id',$user_id], ['date', date('Y-m-d')]])->count();

        $step = (int)($attendanc_cnt/20) +1;
        $this_step_attandanc_cnt = $attendanc_cnt%20;
        if ($this_step_attandanc_cnt == 0) {
            if ($today_ck == 0 ) {
                if ($step != 1) {
                }
            } else {
                $step--;
                $this_step_attandanc_cnt = 20;
            }
        }
        $data['step'] = $step;
        $data['this_step_attandance_cnt'] = $this_step_attandanc_cnt;
        $data['today_ck'] = $today_ck > 1 ? 1 : 0;
        $data['this_setp_block_history'] = Npick_blocks::select('block')
            ->where([['user_id', $user_id],['type','attendance']])
            ->limit($this_step_attandanc_cnt != 20 ? $this_step_attandanc_cnt : 19)
            ->offset(($step-1)*19)
            ->orderBy('created_at', 'ASC')
            ->get();
        if ($this_step_attandanc_cnt == 20) {
            $last = Npick_blocks::select('block')
                ->where([['user_id', $user_id],['type','step_attendance']])
                ->orderBy('created_at', 'DESC')
                ->first();

            $data['this_setp_block_history'][] = $last;

        }
        return response()->json(['success' => true,  'c' =>$user_id, 'alert' => '', 'data' => $data], 200);
    }

    /**
     * 출석 및 N+ 포인트 지급
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function attendance(Request $request)
    {
        $user = Auth()->user();
        if (!$user) {
            return response()->json(['success' => false, 'code' => '', 'alert' => '로그인 후 이용하실 수 있습니다.', 'data' => ''], 200);
        }

        $today_ck = Attendances::where([['user_id',$user->id], ['date', date('Y-m-d')]])->count();

        if ($today_ck > 0) {
            return response()->json(['success' => false, 'code' => '', 'alert' => '이미 출석을 하였습니다.', 'data' => ''], 200);
        } else {
            $default = Settings::where('option_name', 'default_block')->select('option_value')->first();
            $odd = Settings::where('option_name', 'odd_step')->select('option_value')->first();
            $even = Settings::where('option_name', 'even_step')->select('option_value')->first();

            $att = new Attendances();
            $att->user_id = $user->id;
            $att->date = date('Y-m-d');
            $att->created_by = $user->id;
            $att->updated_by = $user->id;
            $result = $att->save();

            if ($result > 0) {
                $attendanc_cnt = Attendances::where([['user_id',$user->id]])->count();

                $step = (int)($attendanc_cnt/20);
                $this_step_attandanc_cnt = $attendanc_cnt%20;
                if ($this_step_attandanc_cnt == 0) {
                    $step_block = Block_steps::select('id', 'step', 'block')
                        ->where('step', $step)
                        ->orderBy('id','DESC')
                        ->first();

                    $ck = Npick_blocks::where([['user_id', $user->id],['type','step_attendance'],['created_at','>=',date('Y-m-d 00:00:00')]])->count();
                    if ($step_block) {
                        if ($step%2 > 0) {
                            if ($ck == 0 ) {
                                $block = new Npick_blocks();
                                $block->user_id = $user->id;
                                $block->block = $step_block->block;
                                $block->type = 'step_attendance';
                                $block->description = '<p><b>' . $step .'회차 출석체크</b>를 완료하여 N+Point를 지급합니다.</p>';
                                $block->created_by = $user->id;
                                $block->updated_by = $user->id;
                                $result = $block->save();
                            }
                        } else {
                            if ($ck == 0 ) {
                                $block = new Npick_blocks();
                                $block->user_id = $user->id;
                                $block->block = $step_block->block;
                                $block->type = 'step_attendance';
                                $block->description = '<p><b>' . $step . '회차 출석체크</b>를 완료하여 N+Point를 지급합니다.</p>';
                                $block->created_by = $user->id;
                                $block->updated_by = $user->id;
                                $result = $block->save();
                            }
                        }
                    } else {
                        if ($step%2 > 0) {
                            if ($ck == 0 ) {
                                $block = new Npick_blocks();
                                $block->user_id = $user->id;
                                $block->block = $odd->option_value;
                                $block->type = 'step_attendance';
                                $block->description = '<p><b>' . $step . '회차 출석체크</b>를 완료하여 N+Point를 지급합니다.</p>';
                                $block->created_by = $user->id;
                                $block->updated_by = $user->id;
                                $result = $block->save();
                            }
                        } else {
                            if ($ck == 0 ) {
                                $block = new Npick_blocks();
                                $block->user_id = $user->id;
                                $block->block = $even->option_value;
                                $block->type = 'step_attendance';
                                $block->description = '<p><b>' . $step . '회차 출석체크</b>를 완료하여 N+Point를 지급합니다.</p>';
                                $block->created_by = $user->id;
                                $block->updated_by = $user->id;
                                $result = $block->save();
                            }
                        }
                    }
                } else {
                    $ck = Npick_blocks::where([['user_id', $user->id],['type','attendance'],['created_at','>=',date('Y-m-d 00:00:00')]])->count();
                    if ($ck == 0 ) {
                        $block = new Npick_blocks();
                        $block->user_id = $user->id;
                        $block->block = $default->option_value;
                        $block->type = 'attendance';
                        $block->description = '<p><b>출석체크</b>를 완료하여 N+Point를 지급합니다.</p>';
                        $block->created_by = $user->id;
                        $block->updated_by = $user->id;
                        $result = $block->save();
                    }
                }

                if ($result > 0) {
                    $data = [];

                    return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
                } else {
                    return response()->json(['success' => false, 'code' => '', 'alert' => '일시적인 오류가 발생 하였습니다.', 'data' => ''], 200);
                }
            } else {
                return response()->json(['success' => false, 'code' => '', 'alert' => '일시적인 오류가 발생 하였습니다.', 'data' => ''], 200);
            }
        }
    }

    /**
     * N+ 포인트 출금신청
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function withdraw(Request $request)
    {
        $user = Auth()->user();

        $wallet = $request->input('wallet');
        $wallet2 = $request->input('wallet2');
        $block = $request->input('block');

        if (!$wallet) {
            return response()->json(['success' => false, 'code' => '', 'alert' => '첫번째 지갑 주소를 입력해주세요.', 'data' => ''], 200);
        }


        if (!$block) {
            return response()->json(['success' => false, 'code' => '', 'alert' => '출금 신청할 N+ point개수를 입력해주세요.', 'data' => ''], 200);
        }

        $npick_block = Npick_blocks::where([['user_id', $user->id]])
            ->selectRaw('SUM(block) as total_block')
            ->first();

        $npick_block = empty($npick_block['total_block']) ? 0 : $npick_block['total_block'];

        if ($npick_block < $block) {
            return response()->json(['success' => false, 'code' => '', 'alert' => '보유하고 있는 N+ point의 수량이 부족합니다.', 'data' => ''], 200);
        }

        DB::beginTransaction();
        $withdraw = new Npick_block_withdraws();
        $withdraw->user_id = $user->id;
        $withdraw->wallet = $wallet;
        $withdraw->npick_block = $block;
        $withdraw->created_by = $user->id;
        $withdraw->updated_by = $user->id;
        $result = $withdraw->save();

        if ($result > 0) {
            $nblock = new Npick_blocks();
            $nblock->user_id = $user->id;
            $nblock->block = -$block;
            $nblock->type = 'withdraw';
            $nblock->description = '<p><b>' . $block .'NPICK</b>을 출금신청 하였습니다.</p>';
            $nblock->created_by = $user->id;
            $nblock->updated_by = $user->id;
            $result2 = $nblock->save();

            if ($result2 > 0) {
                DB::commit();

                $data = [];

                return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
            } else {
                DB::rollBack();

                return response()->json(['success' => false, 'code' => '', 'alert' => '일시적인 오류가 발생 하였습니다.', 'data' => ''], 200);
            }
        } else {
            DB::rollBack();

            return response()->json(['success' => false, 'code' => '', 'alert' => '일시적인 오류가 발생 하였습니다.', 'data' => ''], 200);
        }
    }

    /**
     * N+ 포인트 입금 / 출금 내역 조회
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function view_history(Request $request)
    {
        $user = Auth()->user();

        $history = Npick_blocks::select('id','type','block','description','created_at')
            ->where([['user_id', $user->id]])
            ->orderBy('id','DESC')
            ->get();

        foreach ($history as $hi) {
            if (count(explode('<p>',$hi->description)) == 1) {
                $hi->description = '<p>' . $hi->description . '</p>';
            }
        }
        $data = [];
        $data['history'] = $history;

        return response()->json(['success' => true, 'alert' => '', 'data' => $data], 200);
    }
}
